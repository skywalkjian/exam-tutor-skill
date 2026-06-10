# 执行控制

本参考文件定义考试辅导技能的意图路由、项目工作区初始化、状态恢复、用户确认关卡和覆盖审计。它的目标是让技能像一个可恢复的项目工作流，而不是一次性聊天生成器。学科路径选择由 [domain-router.md](domain-router.md) 负责；本文件只在状态检查和初始化时执行该路由结果。

## 1. 意图路由

开始前先判断用户当前请求属于哪一类，然后只执行必要流程。

| 意图 | 触发表达 | 执行策略 |
|---|---|---|
| 新项目启动 | “下载 skywalkjian/exam-tutor-skill 并执行”“开始一个新的备考项目”“用 exam-tutor 帮我建项目” | 如果没有项目名或学科路径，先同时询问缺失信息；两者都有后创建独立工作区和材料目录，写入等待资料状态，不分析 |
| 资料已放好，开始工作 | “资料放好了”“开始工作”“开始分析”“可以开始了” | 进入对应项目工作区，先运行材料关卡，再按 runtime 继续 |
| 完整构建 | “帮我准备考试”“分析这些材料”“生成备考体系” | 从材料关卡开始，完整生成真题索引、知识图谱、知识点文件、审查和学习计划 |
| 初始化材料目录 | “我还没整理材料”“该怎么放文件” | 创建或说明 `materials/` 结构，不生成考试策略 |
| 增量更新 | “我又加了几份真题/PPT/笔记” | 比对材料指纹，只重跑新增或受影响的分析、KP 和审查 |
| 覆盖审计 | “检查有没有漏题”“帮我 review 这些产物” | 读取 `_analysis/question-index.json`、`knowledgepointslist.md` 和 `knowledge-points/`，生成或更新审查报告 |
| 单点答疑 | “KP-03 看不懂”“这个知识点怎么学” | 读取相关 KP、前置 KP 和材料证据，解释或拆分，不默认重建全项目 |
| 学习计划 | “还有 N 天怎么安排” | 若已有可靠知识图谱和覆盖审计，生成/更新 `study-plan.md`；否则先说明必须分析材料 |

如果意图不清晰，默认做最小安全动作：检查是否已有 exam-tutor 工作区。没有工作区时询问项目名和学科路径；有工作区时检查材料、学科路径和现有状态，并向用户说明下一步需要什么。

## 2. 项目工作区初始化

首次启动新备考项目时，执行以下流程：

1. 如果用户没有同时给出项目名和学科路径，按 [domain-router.md](domain-router.md) 询问缺失信息。默认问题是：“这次备考项目叫什么名字？这门课属于哪条路径：理工科，还是人文社科？”不要先创建目录或分析材料。
2. 用户给出项目名和学科路径后，在当前会话工作目录下创建一个同名工作区。若当前目录是下载来的 `exam-tutor-skill` 仓库或其 `exam-tutor/` skill 目录，在其父目录创建工作区，避免把课程资料混入 skill 源码。
3. 目录名默认使用用户原始项目名；如果名称包含 `/`、`\`、`:`、控制字符或首尾空白，将这些字符替换为 `-` 并折叠连续空白。保留中文和常见课程名字符。
4. 如果目标目录已存在：
   - 若里面已有 `_analysis/exam-tutor-state.json`，视为已有工作区，询问用户是继续这个项目还是换一个名字。
   - 若目录非空但不是 exam-tutor 工作区，询问是否使用该目录，避免覆盖用户文件。
5. 创建当前 track 的 `output-templates.md` 中的项目工作区结构，并写入 `_analysis/exam-tutor-state.json`：
   - `mode`: `waiting-for-materials`
   - `project_name`: 用户提供的项目名
   - `workspace_dir`: 实际创建的目录
   - `discipline_track`: `stem` 或 `humanities-social-sciences`
   - `route.locked`: `true`
   - `route.workflow`: 对应 track 的 `workflow.md`
   - `runtime.mode`: `setup-only`
   - `runtime.execution_strategy`: `serial`
   - `runtime.completed_phases`: `["project_workspace_init"]`
6. 初始化完成后停止，回复用户：工作区已创建、已锁定的学科路径、各材料目录用途、请放入资料、放好后说“开始工作”。

初始化阶段不运行材料分析、不生成 `knowledgepointslist.md`、不生成 KP 文件、不生成学习计划。

## 3. 状态检查

每次开始工作前检查这些路径：

```text
PROJECT.md
materials/
_analysis/exam-tutor-state.json
_analysis/question-index.json
_analysis/coverage-audit.md
knowledgepointslist.md
knowledge-points/
study-plan.md
```

如果这些文件已存在，先判断当前状态：

- 如果状态文件缺少 `discipline_track` 或 `route.locked`，先询问“这个项目应该走哪条路径：理工科，还是人文社科？”补写状态后再继续正式分析。
- `mode` 是 `waiting-for-materials` 且用户未明确要求开始：只提醒用户放入材料并停止。
- `mode` 是 `waiting-for-materials` 且用户说“开始工作”：进入材料关卡。
- 用户明确要求“重新生成全部”时，可以完整重建。
- 只有新增材料时，优先增量更新。
- 只有某个 KP 有问题时，只更新相关 KP、依赖关系和审查结果。
- 现有产物与材料明显不一致时，说明原因并请求确认后重建。

完成状态检查后，按 [runtime/_detect.md](runtime/_detect.md) 选择 runtime mode 和 execution strategy。意图路由决定“做什么”，runtime mode 决定“完整构建、恢复、只审查还是单点修复”，execution strategy 决定“并行还是串行”。学科路径决定“用哪套内容和模板”，且一旦锁定不得混用。

## 4. 状态文件

完整构建、增量更新或审查修复后，更新 `_analysis/exam-tutor-state.json`。字段结构以当前 track 的 `output-templates.md` 中的状态文件模板为准，并保留 `discipline_track` 与 `route`。

这个文件不需要复杂哈希；记录文件路径、大小和修改时间，已经足够支持大多数增量判断。runtime 进度字段由 [runtime/phase-barrier.md](runtime/phase-barrier.md) 定义和解释。

## 5. 真题索引关卡

真题索引是后续所有产物的共同黑板。不要让不同 Agent 自己维护隐含题目列表。

### 5.1 题目 ID 规则

每道题分配稳定 ID：

```text
Q-[来源短名]-[题号]
```

示例：

```text
Q-2025-final-01
Q-2025-final-02b
Q-2024-midterm-03
```

如果文件没有年份或考试名，使用文件名 slug。子题可以用 `02a`、`02b`。

### 5.2 合并后的结构

主流程合并 `_analysis/questions/*.json` 为 `_analysis/question-index.json`，并生成人类可读的 `_analysis/question-index.md`，方便用户审查题目是否漏提或误提。具体字段和 Markdown 结构以当前 track 的 `output-templates.md` 为准。

## 6. 用户确认关卡

这些动作前需要简短确认：

- 覆盖或批量重写已有 `knowledge-points/` 文件。
- 删除或合并已有知识点。
- 将低置信度题目纳入最终学习计划。
- 生成最终版 `study-plan.md`，尤其是剩余时间很短时。

确认时不要长篇解释，只列出：

```markdown
我将更新：
- knowledgepointslist.md：原因
- knowledge-points/KP-03-xxx.md：原因
- _analysis/coverage-audit.md：原因

确认后继续。
```

如果只是创建新的 `_analysis/` 中间文件或补充缺失目录，不需要确认。

## 7. 覆盖审计

生成或更新 `knowledgepointslist.md` 后，立即生成 `_analysis/coverage-audit.md`。

审计必须回答：

- `question-index` 中共有多少题？
- 每道题是否映射到至少一个 KP？
- 每道题是否在某个 KP 文件中有完整解析计划或已完成解析？
- 是否存在低置信度提取、乱码、缺页、图表不可读？
- 是否存在 KP 声称解析了某道题，但题目 ID 不在 `question-index` 中？

如果有未覆盖题：

1. 先判断应新增 KP、拆分 KP，还是把题目映射到已有 KP。
2. 更新 `knowledgepointslist.md`。
3. 更新受影响 KP 文件。
4. 重新运行覆盖审计。

## 8. 可恢复执行

长任务可能被中断。恢复时按这个顺序判断进度：

1. 有初始化状态但无材料：停在 `waiting-for-materials`。
2. 有材料但没有 `question-index.json`：从材料关卡和真题提取开始。
3. 有 `question-index.json` 但没有 `knowledgepointslist.md`：从知识图谱构建继续。
4. 有 `knowledgepointslist.md` 但 `knowledge-points/` 不完整：只生成缺失 KP。
5. 有 KP 文件但没有 `quality-review.md`：从质量审查继续。
6. 有审查报告且列出问题：先修复问题，再生成学习计划。
7. 全部存在：根据用户请求做审计、增量更新或答疑。

如果 `_analysis/exam-tutor-state.json` 中的 `runtime.completed_phases` 与实际文件状态冲突，以实际文件存在性和质量审计结果为准，并在状态文件中记录修正。

## 9. 最小产出原则

不要因为技能功能完整就每次都生成所有文件。根据用户意图选择最小充分产出：

- 用户只问“材料够不够”：只输出材料清单和缺口。
- 用户只说“下载并执行 skill”但没有项目名或学科路径：只询问缺失的项目名和学科路径。
- 用户只完成项目初始化：只创建工作区和目录，等待资料。
- 用户只问“还有 3 天怎么复习”：如果已有知识图谱，只更新计划。
- 用户只问“这道题为什么这么做”：定位题目和 KP，给解释，必要时补写该题解析。
- 用户要求完整备考体系：才执行完整构建。
