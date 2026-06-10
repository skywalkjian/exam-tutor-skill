---
name: exam-tutor-runtime-create-agent
description: 统一的 Agent 创建接口，自动适配 Claude、Codex、Kimi 或串行 fallback。
---

# 统一 Agent 创建接口

task skill 不直接关心底层环境。先由 [`_detect.md`](_detect.md) 选择 host 和 runtime，再按本文件生成 Agent 任务。Agent prompt 必须包含当前 `discipline_track`，并引用同一路径的 workflow/template；不得让 Agent 自行选择或混用路径。

运行时层只规定任务边界、输出路径和阶段顺序。Evidence Agent 的中间分析格式由当前 track 的 `agent-team.md` 决定；不要在运行时层额外要求固定小节、表格、字段顺序或逐项填空格式。

## 任务对象

所有 Agent 任务使用统一字段：

```json
{
  "name": "past-paper-2025-final",
  "role": "question | evidence | writer | review",
  "description": "分析 2025 final 真题",
  "discipline_track": "stem | humanities-social-sciences",
  "track_workflow": "references/tracks/.../workflow.md",
  "source_files": ["materials/past-papers/2025-final.pdf"],
  "source_slug": "2025-final",
  "priority_context": "高频考点或用户目标",
  "output_paths": [
    "_analysis/past-paper-analysis-2025-final.md",
    "_analysis/questions/2025-final.json"
  ],
  "prompt": "完整任务说明"
}
```

父流程在创建任务前必须先生成全局任务表，确保每个 `output_paths` 在本次运行和已有工作区中都唯一。增量更新时优先复用来源文件的 `source_slug`，不要因为排序变化重命名旧输出。

## create_question_agents(question_tasks)

用于 Phase 1，只处理真题、样题、复习题等题目来源。

必须保证：

- 每个真题、样题或复习题文件一个 Question Agent。
- 每个 Question Agent 必须输出 `_analysis/questions/<source-slug>.json`。
- 题目来源分析输出使用 `_analysis/past-paper-analysis-<source-slug>.md` 或当前 track 允许的等价命名。
- `source-slug` 必须从来源文件名稳定生成；不得用容易在增量更新中碰撞的裸序号作为唯一标识。
- Question Agent 不得读取或修改 `knowledgepointslist.md`、`knowledge-points/`、`coverage-audit.md` 或其他 Agent 输出。
- 所有 Question Agent 完成后，父流程必须先执行 Phase 1.5 合并 `question-index`，再创建 Phase 2 Agent。

如果没有题目来源，父流程不创建 Question Agent，直接写入空的 `_analysis/question-index.json` 和 `_analysis/question-index.md`，再进入 Phase 2。

## create_evidence_agents(evidence_tasks)

用于 Phase 2，只处理讲义、课件、笔记、阅读文本、补充材料等非题目证据。

必须保证：

- 每个讲义、教材、阅读文本等大文件一个 Evidence Agent。
- 课件/笔记和补充材料可按类别合并为少量 Agent；材料很大时继续拆分。
- 每个 Evidence Agent 的 prompt 必须包含 `_analysis/question-index.json` 的摘要、高频考点或“无真题”的替代依据。
- Evidence Agent 只能写自己的 `_analysis/*-analysis-<source-slug>.md` 或约定的类别分析文件。
- Evidence Agent 不得直接修改 `knowledgepointslist.md`、`knowledge-points/` 或 `_analysis/coverage-audit.md`。
- Evidence Agent 必须遵守当前 track 的材料分析重点。
- Evidence Agent 的中间分析文件不强制套用固定章节、表格或字段顺序。

## create_writer_agents(kp_tasks)

用于 Phase 4。

每个 KP 一个 Writer Agent。每个任务必须包含：

- KP 编号和主题名。
- `knowledgepointslist.md` 中该 KP 的完整行。
- 该 KP 负责解析的题目 ID 列表。
- `question-index` 中对应题目的完整题干。
- 相关材料摘录。
- 前置 KP 摘要或文件路径。
- 当前 track 的 KP 文件模板和质量标准。

Writer Agent 只写自己的 `knowledge-points/KP-*.md`。

如果 KP 之间存在前置依赖，父流程先把 `knowledgepointslist.md` 中的依赖图分层，并为每个 Writer 提供稳定的前置 KP 摘要。只有当后置 KP 明确需要前置 KP 文件全文且该文件尚未生成时，才按依赖层分批并行：同一层可并行，下一层必须等待上一层完成。

## create_review_agent(review_task)

用于 Phase 5。

输入：

- `_analysis/question-index.json`
- `knowledgepointslist.md`
- `knowledge-points/`
- `_analysis/coverage-audit.md`

输出：

- `_analysis/quality-review.md`
- 更新 `_analysis/coverage-audit.md`

## Runtime 适配

只有 `runtime.execution_strategy=team` 时才读取宿主适配模板。`serial` 时不要创建子 Agent，也不要读取宿主并行适配模板。

### Claude Code

使用 [`claude-team.md`](claude-team.md) 的 Agent Team 模板。

### Codex

使用 [`codex-subagent.md`](codex-subagent.md) 的自然语言 subagent 模板。

### Kimi Code CLI

使用 [`kimi-team.md`](kimi-team.md) 的 `Agent()` + `TaskList/TaskOutput` 模板。

### Fallback / Serial

如果 `runtime.execution_strategy=serial`，按 [`phase-barrier.md`](phase-barrier.md) 串行执行，不跳过任何关键产物。

## 串行 fallback

串行执行时，不创建子 Agent，而是主流程逐个执行任务：

```python
run_material_gate()
for task in question_tasks:
    run_question_task(task)
merge_question_index()

for task in evidence_tasks:
    run_evidence_task(task)
build_knowledge_map()

for writer_layer in dependency_ordered_writer_layers:
    for task in writer_layer:
        run_writer_task(task)

run_review_task(review_task)
ask_timeline_before_final_plan()
```

输出结构必须与 `execution_strategy=team` 时一致。
