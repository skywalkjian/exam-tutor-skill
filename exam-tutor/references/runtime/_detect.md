---
name: exam-tutor-runtime-detect
description: 检测当前 AI 宿主、Agent 能力、用户意图、已有产物、runtime mode 和 execution strategy。
---

# Runtime 检测

exam-tutor 使用三层检测：

1. **Host runtime**：当前执行环境是否支持并行 Agent。
2. **Workflow runtime**：本次任务应初始化项目、等待资料、完整构建、增量更新、审查还是修补。
3. **Execution strategy**：当前阶段实际以 `team` 还是 `serial` 执行。

学科路径不是 runtime mode。路径由 `discipline_track` 决定，只能是 `stem` 或 `humanities-social-sciences`，并由 `references/domain-router.md` 锁定。

## 1. Host Runtime 检测

优先识别当前 AI 宿主。宿主识别只决定优先尝试哪种适配器；是否进入 `team` 必须由真实 Agent 能力探测决定。无法确认能创建并等待子 Agent 时，按不支持并行处理。

```python
import os
import shutil

if os.environ.get("CLAUDE_CODE") == "1" or os.environ.get("CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS"):
    HOST_RUNTIME = "claude"
elif os.environ.get("CODEX") == "1":
    HOST_RUNTIME = "codex"
elif os.environ.get("KIMI_CODE_CLI") == "1" or os.environ.get("KIMI") == "1" or shutil.which("kimi"):
    HOST_RUNTIME = "kimi"
else:
    HOST_RUNTIME = "fallback"

def detect_agent_team_capability(host_runtime):
    if host_runtime == "claude":
        return bool(os.environ.get("CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS"))
    if host_runtime == "codex":
        return probe_native_subagent_tool()  # 必须能创建、等待并读取 subagent 输出；无法探测时返回 False
    if host_runtime == "kimi":
        return probe_kimi_agent_api()        # 必须能调用 Agent/TaskList/TaskOutput；无法探测时返回 False
    return False

AGENT_TEAM_CAPABLE = detect_agent_team_capability(HOST_RUNTIME)
```

| Host | 特征 | Agent 适配器 | 进入 `team` 的条件 |
|---|---|---|
| `claude` | Claude Code | 使用 `claude-team.md` | Agent Team 功能已启用并可调用 |
| `codex` | Codex | 使用 `codex-subagent.md` | native subagent 能力可探测、可等待、可取回输出 |
| `kimi` | Kimi Code CLI | 使用 `kimi-team.md` | `Agent()`、`TaskList`、`TaskOutput` 可调用 |
| `fallback` | 无明确适配器 | 使用 `serial` | 永不进入 `team` |

如果 Host 是 `claude`、`codex` 或 `kimi` 但 `AGENT_TEAM_CAPABLE=False`，不要先进入 `team` 再失败回退；直接选择 `serial`，并在状态文件中记录 host 与降级原因。

## 2. Workflow Mode 检测

Workflow mode 描述用户想做什么。

| Workflow | 触发场景 |
|---|---|
| `project-bootstrap` | 用户要求下载/执行 exam-tutor、开始新备考项目，但尚未建立工作区，或尚未同时提供项目名和学科路径 |
| `waiting-for-materials` | 工作区已创建，用户还在放资料，未要求开始分析 |
| `full-build` | 第一次准备考试、分析整个材料文件夹、生成完整备考体系 |
| `incremental-update` | 新增真题、PPT、笔记、复习范围，只更新受影响内容 |
| `audit` | 检查漏题、review 现有产物、验证一致性 |
| `topic-repair` | 某个 KP 看不懂、某道题解析不清楚、前置知识缺失 |
| `material-init` | 用户还没整理材料，只需要创建或说明材料目录 |

检测规则：

- 如果当前目录及其合理父目录中没有 `_analysis/exam-tutor-state.json`，且用户说“下载并执行”“开始新项目”“创建备考项目”，进入 `project-bootstrap`。
- 如果用户没有提供项目名或学科路径，停在 `project-bootstrap` 并按 `references/domain-router.md` 询问缺失信息。
- 如果已有状态文件但缺少 `discipline_track` 或 `route.locked`，先补问路径并写回状态，再选择正式 runtime。
- 如果状态文件 `mode` 是 `waiting-for-materials`，且用户没有说“开始工作/开始分析/资料放好了”，保持 `waiting-for-materials`。
- 如果状态文件 `mode` 是 `waiting-for-materials`，且用户说“开始工作/开始分析/资料放好了”，进入 `full-build` 并从 `material_gate` 开始。

## 3. Runtime Mode 与 Execution Strategy 选择

Runtime mode 描述本次任务类型；execution strategy 描述当前阶段怎么执行。`resume` 不是串行或并行策略，恢复后仍要按当前宿主能力和待处理任务规模选择 `team` 或 `serial`。

```python
def choose_execution_strategy(agent_team_capable, task_count, category_count):
    if agent_team_capable and (task_count >= 3 or category_count >= 2):
        return "team"
    return "serial"

def choose_runtime(user_intent, host_runtime, agent_team_capable, material_count, category_count, state_exists, partial_outputs_exist, state_mode=None, pending_task_count=None):
    task_count = pending_task_count if pending_task_count is not None else material_count
    if user_intent in ("project-bootstrap", "waiting-for-materials", "material-init"):
        return {"mode": "setup-only", "execution_strategy": "serial"}
    if user_intent == "audit":
        return {"mode": "audit-only", "execution_strategy": "serial"}
    if user_intent == "topic-repair":
        return {"mode": "topic-repair", "execution_strategy": "serial"}
    if user_intent == "incremental-update":
        return {
            "mode": "resume",
            "execution_strategy": choose_execution_strategy(agent_team_capable, task_count, category_count)
        }
    if user_intent == "full-build" and state_mode == "waiting-for-materials":
        return {
            "mode": "full-build",
            "execution_strategy": choose_execution_strategy(agent_team_capable, task_count, category_count)
        }
    if state_exists or partial_outputs_exist:
        return {
            "mode": "resume",
            "execution_strategy": choose_execution_strategy(agent_team_capable, task_count, category_count)
        }
    return {
        "mode": "full-build",
        "execution_strategy": choose_execution_strategy(agent_team_capable, task_count, category_count)
    }
```

恢复或增量更新时，`task_count` 应使用本次待处理任务数，而不是工作区全部材料数；这样小范围修补不会因为历史材料很多而误触发团队模式。

| Runtime | 含义 |
|---|---|
| `setup-only` | 只创建或维护项目工作区和材料目录，不分析材料 |
| `full-build` | 从材料关卡开始完整构建关键产物 |
| `resume` | 从已有 `_analysis/` 状态继续 |
| `audit-only` | 只审查现有产物，不重读全量材料 |
| `topic-repair` | 只修补一个知识点、一道题或一条前置链 |

| Execution strategy | 含义 |
|---|---|
| `team` | 同一阶段内创建多个 Agent 并行执行 |
| `serial` | 不创建子 Agent，由主流程逐个执行同一批任务 |

## 4. 状态检查

选择 runtime 前，检查：

```text
PROJECT.md
materials/
_analysis/exam-tutor-state.json
_analysis/questions/
_analysis/question-index.json
_analysis/coverage-audit.md
_analysis/quality-review.md
knowledgepointslist.md
knowledge-points/
study-plan.md
```

如果 `_analysis/exam-tutor-state.json` 与实际文件状态冲突，以实际文件和质量审计结果为准，并更新状态文件。

## 5. 写入状态文件

每次确定 runtime 后，写入或更新：

```json
{
  "runtime": {
    "host": "claude | codex | kimi | fallback",
    "mode": "setup-only | full-build | resume | audit-only | topic-repair",
    "execution_strategy": "team | serial",
    "agent_team_capable": true,
    "fallback_reason": "",
    "completed_phases": [],
    "last_phase": "",
    "reused_outputs": [],
    "pending_outputs": []
  }
}
```

记录 host 是为了后续调试；记录 mode 和 phase 是为了恢复执行。路径字段必须保留在 runtime 外层：

```json
{
  "discipline_track": "stem | humanities-social-sciences",
  "route": {
    "locked": true,
    "workflow": "references/tracks/stem/workflow.md | references/tracks/humanities-social-sciences/workflow.md"
  }
}
```
