# Agent 团队路由

本文件是兼容入口，不再提供具体学科路径的 Agent 分工。

使用团队模式前，必须先按 [domain-router.md](domain-router.md) 锁定 `discipline_track`，并确认 `runtime.execution_strategy=team`，然后只读取对应路径的团队说明：

- 理工科：`references/tracks/stem/agent-team.md`
- 人文社科：`references/tracks/humanities-social-sciences/agent-team.md`

## 公共约束

- `team` execution strategy 只决定是否并行，不决定学科写法。
- 子 Agent 共享 `_analysis/` 下的黑板产物，但最终输出结构由当前 track 的 `output-templates.md` 决定。
- 不允许在人文社科项目中使用理工科 Writer 指令，也不允许在理工科项目中使用人文社科专题模板。
- 如果路径模板尚未填充完整，说明缺口并按当前路径的最小骨架执行，不要借用另一条路径。
