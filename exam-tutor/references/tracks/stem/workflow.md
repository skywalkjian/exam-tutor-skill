# 理工科路径 Workflow

本路径只用于 `discipline_track=stem` 的项目。进入本文件后，不要读取或借用 `references/tracks/humanities-social-sciences/` 下的任何内容。

## 必读文件

- 输出模板：[output-templates.md](output-templates.md)
- 团队模式：[agent-team.md](agent-team.md)
- 公共材料处理：../../material-processing.md
- 公共执行控制：../../execution-control.md
- 公共 runtime：../../runtime/index.md

## 目标学习者

- 时间紧迫，需要准备以计算、推导、证明、建模、实验设计或概念应用为核心的大学考试。
- 可能从接近零基础开始，需要先补前置概念，再能做题。
- 优先考虑考试表现、可靠覆盖和高效分流，而非学科全覆盖。

## 理工科核心原则

- 备考不等于学习；考试是一场策略博弈。优化目标是通过考试并最大化分数。
- 真题是最高优先级证据。每道真题必须逐字进入 `_analysis/question-index.json`，并映射到至少一个 KP。
- 知识点文件要能从零教会学生：直觉、定义、符号、公式、推导、基础案例、完整例题、题型流程、真题解析都要闭环。
- 公式第一次出现时解释每个符号、适用条件、为什么这样写、何时有效、何时无效和常见误用。
- 繁重计算使用代码工具完成；不要用心算冒险。
- Mermaid 图用于概念依赖、证明结构、判别流程和题型分支。
- 如果一个文件像速查表而不是迷你课，视为未完成。

## 执行阶段

阶段顺序由公共 [../../runtime/phase-barrier.md](../../runtime/phase-barrier.md) 控制；本路径只定义每个阶段的理工科内容要求。

### Phase 0. 材料关卡

- 扫描 `materials/` 下所有课程材料。
- 优先级：历年真题 > 老师复习范围 > 讲义/PPT/课堂笔记 > 补充材料。
- 没有真实课程材料时停止，不生成通用学习建议。
- PDF、PPT、Word、Markdown 的提取按公共材料处理文件执行。

### Phase 1. 真题提取

- 每道题必须逐字提取，包括数值、条件、子问题、图表说明和分值。
- 为每道题建立稳定 ID，并保存到 `_analysis/questions/*.json`。
- 初步标注考查知识点、题型、难度、可能陷阱和提取置信度。
- 如果没有真题，`question-index.json` 保持空数组，不得编造题目。

### Phase 1.5. 真题索引合并

- 合并 `_analysis/questions/*.json` 为 `_analysis/question-index.json`。
- 生成人类可读的 `_analysis/question-index.md`。
- 后续所有 KP、覆盖审计和学习计划都引用该索引，不另起题目列表。

### Phase 2. 材料证据分析

- 讲义分析要提取定义、定理、公式、推导线索、例题、习题、图表含义和章节依赖。
- 课件/笔记分析要提取老师强调、课堂直觉解释、常见易错点、速记口诀和复习提示。
- 有真题时，用真题高频考点引导讲义深挖；没有真题时，用考试说明和课程总结引导。

### Phase 3. 知识图谱

- 生成 `knowledgepointslist.md`，按教学顺序和前置依赖组织 KP。
- 每个 KP 标为：考试关键、前置或辅助。
- 每道真题必须映射到至少一个 KP；未覆盖时新增、拆分或合并 KP。
- 同步生成 `_analysis/coverage-audit.md`。

### Phase 4. 知识点文件

每个 KP 写成一堂从零开始的迷你课。结构和细节以 [output-templates.md](output-templates.md) 为准，最低要求包括：

- `[!tip] 学习指南`
- 课程定位、材料证据、学习目标、前置知识
- Motivation
- 概念关系图
- 从零直觉
- 定义、符号与对象
- 基础案例
- 严格推导 / 原理解释
- 定理、公式与技术工具
- 完整例题
- 题型通用解题流程
- 真题完整解析
- 易错点、快速自检、考前速记

### Phase 5. 质量审查

审查项：

- 真题覆盖完整性
- 题目 ID 与 `question-index.json` 一致性
- 材料引用准确性
- 前置依赖无矛盾、无循环
- 从零可学性
- 公式和符号解释是否完整
- 推导、例题、题型流程和真题解析是否足够支撑考试作答
- 是否存在薄文件

### Phase 6. 学习计划

- 只在用户提供剩余天数或考试日期后生成 `study-plan.md`。
- 从高收益 KP、前置瓶颈和真题题型倒推安排。
- 如果没有真题或提取置信度低，计划中必须标注风险。

## 团队模式

当 `runtime.execution_strategy=team` 时，使用本路径的 [agent-team.md](agent-team.md)。少量材料、宿主不支持并行或无法确认并行能力时，用串行方式执行同一阶段和输出契约。
