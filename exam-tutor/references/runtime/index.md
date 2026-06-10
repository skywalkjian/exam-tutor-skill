---
name: exam-tutor-runtime
description: Exam Tutor runtime 入口索引，按 AutoPku 风格拆分运行环境检测、Agent 创建、平台适配、阶段屏障和恢复规则。
---

# Runtime 索引

本目录定义 exam-tutor 的运行时系统。它借鉴 AutoPku 的 runtime 分层方式，但适配考试辅导场景：先创建独立项目工作区、锁定学科路径并等待材料，之后真题索引、覆盖审计、知识点文件生成和学习计划必须按阶段屏障推进。

## 使用顺序

1. 读取 [`_detect.md`](_detect.md)：识别当前 AI 宿主、用户意图、已有状态和执行模式。
2. 读取 [`phase-barrier.md`](phase-barrier.md)：确认必须遵守的阶段顺序和恢复点。
3. 确认 `_analysis/exam-tutor-state.json` 中已有 `discipline_track` 和 `route.locked=true`；若缺失，回到 `references/domain-router.md` 补问路径。
4. 读取 [`create-agent.md`](create-agent.md)：按统一契约创建 Question、Evidence、Writer、Review agents。
5. 如果 `runtime.execution_strategy=team`，按宿主环境补充读取：
   - Claude Code: [`claude-team.md`](claude-team.md)
   - Codex: [`codex-subagent.md`](codex-subagent.md)
   - Kimi Code CLI: [`kimi-team.md`](kimi-team.md)
6. 如果只是初始化项目或等待材料，使用 `setup-only` 并停止；如果无法确认宿主是否支持并行 Agent，使用 `serial` fallback，跳过宿主并行适配模板，但仍然生成同一 track 的关键产物。

## Runtime 文件索引

| 文件 | 作用 |
|---|---|
| `_detect.md` | 检测 AI 宿主、workflow mode、runtime mode 和 execution strategy |
| `phase-barrier.md` | 定义 `material_gate` 到 `final_plan` 的阶段屏障、resume 和增量更新 |
| `create-agent.md` | 定义 `create_question_agents`、`create_evidence_agents`、`create_writer_agents`、`create_review_agent` 的统一接口 |
| `claude-team.md` | Claude Code Agent Team 调用模板 |
| `codex-subagent.md` | Codex subagent 调用模板 |
| `kimi-team.md` | Kimi Code CLI Agent 调用模板 |

## 核心原则

- Runtime mode 决定“做哪类执行”，execution strategy 决定“并行还是串行”，`discipline_track` 决定“用哪套内容和模板”。
- 新项目先用 `setup-only` 创建工作区，用户放入资料并说“开始工作”后才进入正式分析。
- 阶段内部可以并行，阶段之间必须等待屏障产物完成；Phase 1.5 的 `question-index` 生成前，不启动 Phase 2 证据分析 Agent。
- 主流程负责合并真题索引、知识图谱、覆盖审计和状态文件。
- 子 Agent 只能写自己被分配的中间文件或单个 KP 文件。
- `serial` 模式不降低当前 track 的输出契约，只降低并行度。
