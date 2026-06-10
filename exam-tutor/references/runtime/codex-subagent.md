---
name: exam-tutor-runtime-codex
description: Codex subagent 运行时接口，适配 exam-tutor 的并行分析、知识点撰写和审查流程。
---

# Codex Subagent Runtime

在 Codex 环境中，使用自然语言描述并行 subagent 任务。每个 subagent 必须有明确输入、输出路径和写入边界。运行时模板不要额外要求 Evidence Agent 的中间分析使用固定小节、表格、字段顺序或逐项填空。

## 并行 Question Subagents

```
请并行创建 subagents 处理以下 exam-tutor question tasks。

规则：
- 所有 subagent 必须使用当前 discipline_track 对应的 workflow 和 output-templates。
- 每个 subagent 只处理自己分配的文件。
- 每个 subagent 只写指定 output_paths。
- 每个 question subagent 必须输出 _analysis/questions/<source-slug>.json。
- 不要修改 knowledgepointslist.md、knowledge-points/ 或其他 subagent 的输出。

任务列表：
{question_tasks}
```

所有 question subagents 完成后，父流程先合并 `_analysis/question-index.json` 和 `_analysis/question-index.md`。合并完成前不要创建 evidence subagents。

## 并行 Evidence Subagents

```
请并行创建 subagents 处理以下 exam-tutor evidence tasks。

规则：
- 所有 subagent 必须使用当前 discipline_track 对应的 workflow 和 output-templates。
- 每个 subagent 必须读取父流程提供的 question-index 摘要、高频考点或“无真题”的替代依据。
- 每个 subagent 只处理自己分配的讲义、课件、笔记、阅读材料或补充材料。
- 中间分析文件可由 subagent 自主组织，不要求固定章节、表格或字段顺序；重点是保留足够多的材料细节、关系和判断。
- 每个 subagent 只写指定 output_paths。
- 不要修改 question-index、knowledgepointslist.md、knowledge-points/ 或 coverage-audit.md。

任务列表：
{evidence_tasks}
```

## 并行 Writer Subagents

```
请并行创建 subagents 生成以下知识点文件。

每个 subagent 输入：
- 对应 KP 行
- 对应题目 ID 和完整题干
- 相关材料证据
- 前置 KP 摘要
- 当前 discipline_track 对应的 KP 文件模板和质量标准

每个 subagent 输出：
- 一个 knowledge-points/KP-*.md 文件

规则：
- 真题解析标题必须包含题目 ID。
- 不得新增 question-index 中不存在的“真题”。
- 不得套用另一条学科路径的模板标题或质量标准。
- 不得修改其他 KP 文件。

任务列表：
{kp_tasks}
```

## Review Subagent

```
请创建一个 review subagent。

输入：
- _analysis/question-index.json
- knowledgepointslist.md
- knowledge-points/
- _analysis/coverage-audit.md

输出：
- _analysis/quality-review.md
- 更新 _analysis/coverage-audit.md

检查：
- 每道题是否被至少一个 KP 解析
- KP 文件中的题目 ID 是否都存在于 question-index
- 材料证据是否真实
- 依赖关系是否无循环
- 是否符合当前 discipline_track 的输出模板和质量标准
```

## Codex 执行规则

- 父流程负责合并 `question-index`，不交给 subagent。
- 父流程负责写 `_analysis/exam-tutor-state.json`。
- 如果 subagent 能力不可用，启动前直接使用 `serial`；运行中失败时记录失败任务、已完成输出和降级原因，再按 `resume` + `serial` 继续。
