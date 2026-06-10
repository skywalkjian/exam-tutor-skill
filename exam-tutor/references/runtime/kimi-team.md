---
name: exam-tutor-runtime-kimi
description: Kimi Code CLI Agent Team 运行时接口，适配 exam-tutor 的分阶段任务。
---

# Kimi Code CLI Runtime

Kimi Code CLI 使用 `Agent()` 创建子代理，结果直接返回给父代理；子代理之间不直接通信。父流程负责收集结果、合并索引和推进阶段屏障。

## 创建 Question / Evidence Agents

```python
for task in question_tasks:
    Agent({
        "description": task["description"],
        "prompt": task["prompt"],
        "subagent_type": "coder"
    })

# 等待 question_tasks 完成并合并 question-index 后，再创建 evidence_tasks。
for task in evidence_tasks:
    Agent({
        "description": task["description"],
        "prompt": task["prompt"],
        "subagent_type": "coder"
    })
```

题目提取和材料证据分析任务推荐 `subagent_type: "coder"`。材料探查可以使用 `explore`，但最终提取和写文件必须由 `coder` 完成。

每个任务 prompt 必须写明当前 `discipline_track` 和对应 track workflow，防止子代理自行混用模板。Evidence Agent 的 prompt 必须包含父流程合并后的 `question-index` 摘要。Evidence Agent 的中间分析格式遵守当前 track 的 `agent-team.md`，不由运行时层强制固定小节、表格或字段顺序。

## 后台任务

如果某个材料分析耗时很长，可以后台运行：

```python
Agent({
    "description": "分析大型讲义文件",
    "prompt": "...",
    "subagent_type": "coder",
    "run_in_background": True
})

TaskList({"active_only": True})
TaskOutput({"task_id": "<task_id>"})
```

## 结果收集

Kimi 没有 `SendMessage()` 风格的 Agent 间通信。父流程必须：

1. 等待本阶段所有 Agent 完成。
2. 检查每个 Agent 声明的输出文件是否存在。
3. Question 阶段完成后合并 `_analysis/questions/*.json`。
4. 合并完成后再启动 Evidence 阶段。
5. 推进到下一阶段。

## Writer Agents

```python
for task in kp_tasks:
    Agent({
        "description": f"撰写 {task['kp_id']} {task['topic']}",
        "prompt": task["prompt"],
        "subagent_type": "coder"
    })
```

每个 Writer Agent 只写一个 KP 文件。Writer prompt 必须引用当前 track 的 `output-templates.md`。理工科和人文社科的 KP 文件结构不同，不得互相套用。

## Review Agent

```python
Agent({
    "description": "审查 exam-tutor 输出覆盖和一致性",
    "prompt": review_prompt,
    "subagent_type": "coder"
})
```

Review Agent 输出 `_analysis/quality-review.md` 并更新 `_analysis/coverage-audit.md`。

## Kimi 执行规则

- 子代理结果通过父代理收集。
- 不让子代理直接协调其他子代理。
- 阶段屏障由父流程执行。
- 如果后台任务失败或超时，记录到状态文件并切换为 `resume` 或 `serial`。
