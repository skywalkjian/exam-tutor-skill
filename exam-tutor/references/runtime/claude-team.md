---
name: exam-tutor-runtime-claude
description: Claude Code Agent Team 运行时接口，适配 exam-tutor 的分阶段备考工作流。
---

# Claude Code Agent Team Runtime

在 Claude Code 且 Agent Team 可用时，使用本模板创建并行 Agent。

## 创建 Question / Evidence Agents

```python
for task in question_tasks:
    Agent({
        "name": task["name"],
        "description": task["description"],
        "prompt": task["prompt"]
    })

# 等待所有 question_tasks 完成后，父流程先合并 question-index，再创建 evidence_tasks。
for task in evidence_tasks:
    Agent({
        "name": task["name"],
        "description": task["description"],
        "prompt": task["prompt"]
    })
```

每个 prompt 必须明确：

- 当前 `discipline_track` 和对应 track workflow。
- 输入文件列表。
- 输出文件路径。
- 是否需要输出 `_analysis/questions/*.json`。
- 不得修改最终索引和其他 Agent 输出。
- Question Agent 与 Evidence Agent 分阶段创建；Evidence Agent 的 prompt 必须包含父流程合并后的 `question-index` 摘要。Evidence Agent 的中间分析文件不要求固定章节、表格或字段顺序。

## Phase 1 示例：真题分析

```python
Agent({
    "name": "past-paper-2025-final",
    "description": "逐题提取 2025 final 真题",
    "prompt": """
你是 exam-tutor 的真题分析 Agent。

输入：
- materials/past-papers/2025-final.pdf

输出：
- _analysis/past-paper-analysis-2025-final.md
- _analysis/questions/2025-final.json

要求：
1. 逐字提取每道题，不概括、不改写。
2. 为每道题分配稳定 ID，例如 Q-2025-final-01。
3. 按当前 track 的 workflow 标注初步专题/知识点、题型、难度、分值和提取置信度。
4. 如果题目不可读，标记低置信度，不要猜。
5. 不要修改 knowledgepointslist.md 或 knowledge-points/。
"""
})
```

## Phase 4 示例：理工科知识点 Writer

```python
Agent({
    "name": "writer-KP-02-bayes-theorem",
    "description": "撰写 KP-02 贝叶斯定理",
    "prompt": """
你是 exam-tutor 的知识点撰写 Agent。

输入：
- knowledgepointslist.md 中 KP-02 的完整行
- _analysis/question-index.json 中分配给 KP-02 的题目
- 相关讲义/课件/笔记摘录
- KP-01 条件概率的摘要

输出：
- knowledge-points/KP-02-bayes-theorem.md

要求：
1. 严格遵循 references/tracks/stem/output-templates.md 的 KP 文件模板。
2. 每个真题解析标题必须保留题目 ID。
3. 提炼题型通用解题流程。
4. 只写自己的 KP 文件，不修改其他文件。
"""
})
```

人文社科路径的 Writer prompt 必须改用 `references/tracks/humanities-social-sciences/output-templates.md`，输出专题讲义、概念界定、理论谱系、争议结构、案例材料和自拟题目分析，不要套用上面的理工科示例标题。

## Phase Barrier

Claude Team 可以并行启动同一阶段的多个 Agent，但父流程必须等待本阶段所有输出文件存在并通过基本检查后，再进入下一阶段。

必须等待：

- 所有 `_analysis/questions/*.json` 完成后再合并 `question-index`，合并完成前不创建 Evidence Agents。
- `coverage-audit.md` 完成后再启动 Writer Agents。
- 所有 KP 文件完成后再启动 Review Agent。
