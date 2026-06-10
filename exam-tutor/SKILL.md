---
name: "exam-tutor"
description: "帮助时间紧迫的大学生高效备考大学考试。首次启动新备考项目时，必须同时询问项目名称和学科路径（理工科 / 人文社科），创建独立工作区并锁定路径；后续根据路径只读取对应 track 的 skill 内容，理工科与人文社科两条路径完全隔离。当用户提到考试复习、备考、期末考试、下载并执行 exam-tutor、创建备考项目或需要分析课程材料时使用。"
---

# 考试辅导路由器

`exam-tutor` 是一个项目化备考工作流。主入口只负责公共层：识别意图、创建工作区、锁定学科路径、检查材料、选择 runtime、恢复状态。真正的知识组织、教学文件模板、Agent 分工和质量审查规则必须由已锁定的学科路径提供。

默认使用中文撰写技能说明和生成文件。专业术语、原文概念、公式、作者名、文本名和课程原始措辞应按材料保留。

## 必读顺序

1. 先读 [references/domain-router.md](references/domain-router.md)，确定项目名称和学科路径。
2. 需要处理材料格式时读 [references/material-processing.md](references/material-processing.md)。
3. 需要判断意图、状态恢复或确认关卡时读 [references/execution-control.md](references/execution-control.md)。
4. 需要选择 runtime 时读 [references/runtime/index.md](references/runtime/index.md) 和对应 runtime 参考。
5. 只在路径锁定后读取对应 track：
   - 理工科：读 [references/tracks/stem/workflow.md](references/tracks/stem/workflow.md)。
   - 人文社科：读 [references/tracks/humanities-social-sciences/workflow.md](references/tracks/humanities-social-sciences/workflow.md)。

根目录下的 `references/output-templates.md` 和 `references/agent-team.md` 只是兼容入口，不提供最终模板。最终模板和 Agent 规则必须来自对应 track。

## 路由硬约束

- 新项目启动时，如果用户没有同时给出项目名称和学科路径，必须一次性询问：“这次备考项目叫什么名字？这门课属于哪条路径：理工科，还是人文社科？”
- 如果用户只给了项目名，继续追问学科路径；如果只给了路径，继续追问项目名。
- 创建工作区时，把路径写入 `_analysis/exam-tutor-state.json` 和 `PROJECT.md`。字段使用：
  - `discipline_track`: `stem` 或 `humanities-social-sciences`
  - `route.locked`: `true`
  - `route.workflow`: 对应 track 的 workflow 路径
- 已有工作区如果已经锁定路径，后续不再重新询问，也不要自动切换路径。
- 已有工作区如果缺少路径字段，在正式分析前先补问一次路径；补写状态文件后再继续。
- 路径锁定后，严禁读取、借用或混合另一条路径的 workflow、模板、Agent 分工和质量标准。
- 如果用户要求切换路径，优先创建新工作区或执行显式迁移；不要在原工作区里混合两套产物。

## 共同目标

- 材料为王。没有真实课程材料，不生成看似自信的通用备考策略。
- 真题、样题、考试说明和老师划定范围优先于模型常识。
- 每门课使用独立项目工作区。
- 所有可恢复进度写入 `_analysis/exam-tutor-state.json`。
- 根据用户意图选择最小充分产出，避免无意义地重建全部文件。
- 最终产物必须可审计：知识点列表、材料证据、题目索引、覆盖审查、知识点文件和学习计划之间要能相互追踪。

## 执行入口

先按 [references/domain-router.md](references/domain-router.md) 完成项目名和路径路由，再按 [references/execution-control.md](references/execution-control.md) 判断意图。

项目初始化和等待资料时使用 `setup-only`。正式分析时，先根据用户意图选择 runtime mode（`full-build`、`resume`、`audit-only`、`topic-repair`），再根据宿主 Agent 能力和任务规模选择 execution strategy（`team` 或 `serial`）。检测规则见 [references/runtime/_detect.md](references/runtime/_detect.md)，阶段顺序、屏障、恢复和增量更新规则见 [references/runtime/phase-barrier.md](references/runtime/phase-barrier.md)。

只有在 `execution_strategy=team` 时，才读取对应 track 的 Agent 团队说明。少量材料、不可并行或无法确认并行能力时使用 `serial`，但仍遵守同一 track 的输出契约。

## 配套技能

- 如果材料包含 PDF，并且 `$pdf` 可用，在分析 PDF 前先调用 `$pdf`；不可用时按 [references/material-processing.md](references/material-processing.md) 的内置 PDF 工作流处理。
- 繁重的数学计算、矩阵运算、数值计算、方程求解等只属于理工科路径；进入理工科 workflow 后再决定是否使用 Python、numpy 或 sympy。

## 输出预期

- 初始化阶段只创建项目工作区和材料目录，然后停止等待资料。
- 正式分析阶段产出具体文件，而不是只在聊天里给建议。
- 产物结构、Markdown 模板、质量审查标准以锁定 track 的 `output-templates.md` 和 `workflow.md` 为准。
- 如果材料缺失、提取质量低或无法确认考试范围，明确标注风险，不要假装确定。
