---
name: exam-tutor-runtime-phase-barrier
description: 定义 exam-tutor 的阶段屏障、resume 规则和增量更新顺序。
---

# 阶段屏障

`setup-only` 只完成项目初始化并停止。无论 execution strategy 是 `team` 还是 `serial`，正式分析都按同一阶段顺序推进。`team` 可以在阶段内部并行，但不能跨过屏障提前写最终文件。

## Phase 定义

```text
Phase -1: project_workspace_init
  输入：用户提供的项目名称和学科路径
  输出：<project-name>/ 工作区、PROJECT.md、materials/、_analysis/exam-tutor-state.json，且写入 discipline_track 与 route

Phase 0: material_gate
  输入：materials/ 或用户提供的材料路径
  输出：材料清单、缺失材料说明、目录映射

Phase 1: past_paper_extraction
  输入：materials/past-papers/
  输出：_analysis/past-paper-analysis-*.md、_analysis/questions/*.json

Phase 1.5: question_index_merge
  输入：_analysis/questions/*.json
  输出：_analysis/question-index.json、_analysis/question-index.md

Phase 2: evidence_analysis
  输入：question-index、高频考点、lectures/、notes/、supplements/
  输出：_analysis/lecture-analysis-*.md、slides-notes-analysis.md、supplement-analysis.md

Phase 3: knowledge_map
  输入：question-index + 所有材料分析
  输出：按当前 track 模板生成的 knowledgepointslist.md、_analysis/coverage-audit.md

Phase 4: topic_writing
  输入：knowledgepointslist、question-index、相关材料摘录
  输出：按当前 track 模板生成的 knowledge-points/KP-*.md

Phase 5: quality_review
  输入：question-index、knowledgepointslist、knowledge-points/
  输出：_analysis/quality-review.md、更新后的 _analysis/coverage-audit.md

Phase 6: final_plan
  输入：时间线、覆盖审计、知识点优先级
  输出：study-plan.md、_analysis/exam-tutor-state.json
```

## 标准任务队列

正式分析必须按下面的任务队列推进，不能把后续阶段的 Agent 混入前一阶段：

```text
Phase 0 material_gate
  -> 生成材料清单、材料分类和待处理任务表

Phase 1 question_tasks
  -> 只处理真题、样题、复习题等题目来源
  -> 输出 _analysis/questions/*.json 和题目来源分析

Phase 1.5 question_index_merge
  -> 父流程合并题目片段
  -> 即使没有真题，也写入 {"questions": []}

Phase 2 evidence_tasks
  -> 在 question-index 和高频/低置信度题目信息存在后，处理讲义、课件、笔记、补充材料

Phase 3 knowledge_map
  -> 父流程生成知识图谱和覆盖审计

Phase 4 writer_tasks
  -> 生成 KP 文件；可以按依赖层并行

Phase 5 review_task
  -> 审查覆盖、一致性和模板合规

Phase 6 final_plan
  -> 只在用户时间线明确后生成或更新学习计划
```

`team` 与 `serial` 只改变同一阶段任务的执行方式，不改变任务队列顺序。恢复执行时，先定位缺失的最早屏障产物，再对该阶段重新选择 execution strategy。

## 屏障规则

- Phase 1.5 完成前，不创建 `knowledgepointslist.md`。
- Phase 1.5 完成前，不启动 Phase 2 的讲义、课件、笔记或补充材料 Agent；没有真题时也必须先生成空的 `question-index`。
- 未锁定 `discipline_track` 前，不进入 Phase 0，也不读取任何 track 的输出模板。
- `waiting-for-materials` 状态下，用户未明确说“开始工作”前，不进入 Phase 0。
- Phase 3 完成前，不启动知识点撰写 Agent。
- Phase 5 通过前，不生成无风险标注的最终 `study-plan.md`。
- 如果某阶段发现高风险缺口，先修复该阶段输入，再继续下一阶段。
- 路径锁定后，所有 Phase 3 到 Phase 6 的最终产物都必须使用同一 track；不得跨路径混用模板。

## Resume 规则

| 已有产物 | 下一步 |
|---|---|
| 无工作区、无项目名或无学科路径 | 询问缺失的项目名或学科路径，然后执行 Phase -1 |
| 有项目工作区但状态缺少 `discipline_track` | 询问路径并补写状态，然后从对应阶段继续 |
| 有项目工作区但 `mode=waiting-for-materials` 且材料为空 | 停止，提醒用户放入资料 |
| 有项目工作区且用户说“开始工作” | 从 Phase 0 开始 |
| 只有材料目录 | 从 Phase 0 开始 |
| 有 `_analysis/questions/*.json`，无 `question-index.json` | 从 Phase 1.5 开始 |
| 有 `question-index.json`，无 `knowledgepointslist.md` | 从 Phase 2 或 Phase 3 开始，取决于材料分析是否完整 |
| 有 `knowledgepointslist.md`，KP 文件不完整 | 从 Phase 4 生成缺失 KP |
| 有 KP 文件，无 `quality-review.md` | 从 Phase 5 开始 |
| 有审查报告且列出问题 | 先进入 `topic-repair` 修复问题，再回到 Phase 5 |
| 全部存在 | 按用户意图进入 `audit-only`、`topic-repair` 或增量更新 |

恢复时必须记录：

- 哪些阶段被复用。
- 哪些阶段被重跑。
- 哪些文件将被更新。
- 本次恢复阶段使用的 `execution_strategy`，以及选择 `serial` 的降级原因（如宿主无并行 Agent 能力）。

## 增量更新

新增材料时，不默认全量重建。

1. 比对 `_analysis/exam-tutor-state.json` 中的材料列表。
2. 找出新增、删除或修改的文件。
3. 对新增真题运行 Phase 1，并合并到 `question-index`。
4. 对新增讲义/课件运行 Phase 2。
5. 更新受影响 KP 映射和 `coverage-audit.md`。
6. 只重写受影响的 KP 文件。
7. 重新运行质量审查。
8. 更新 `study-plan.md` 和状态文件。

如果新增真题引入大量新知识点，先说明影响范围，并在批量新增或重写 KP 前请求用户确认。

## 状态字段

每次完成阶段后更新：

```json
{
  "runtime": {
    "mode": "full-build | resume | audit-only | topic-repair | setup-only",
    "execution_strategy": "team | serial",
    "completed_phases": ["project_workspace_init", "material_gate", "past_paper_extraction"],
    "last_phase": "question_index_merge",
    "reused_outputs": [],
    "pending_outputs": []
  }
}
```
