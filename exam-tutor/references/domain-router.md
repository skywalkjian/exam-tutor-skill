# 学科路径路由

本文件定义 `exam-tutor` 的公共路由层。它只解决“这是哪个项目、走哪条学科路径、状态如何锁定”的问题，不定义具体知识点写法。

## 1. 路径定义

| 用户可见选项 | 状态值 | 读取的 workflow | 适用课程 |
| --- | --- | --- | --- |
| 理工科 | `stem` | `references/tracks/stem/workflow.md` | 数学、物理、计算机、工程、统计、经济计量、偏公式推导和计算题的课程 |
| 人文社科 | `humanities-social-sciences` | `references/tracks/humanities-social-sciences/workflow.md` | 文史哲、政治学、社会学、法学、传播、教育、艺术理论、偏概念论述和材料分析的课程 |

不要根据课程名自作主张地隐藏选择。即使课程看起来明显，也要在新项目初始化时询问路径，因为同名课程的考试形态可能不同。

## 2. 新项目询问规则

新项目启动时必须同时收集两个信息：

```text
这次备考项目叫什么名字？这门课属于哪条路径：理工科，还是人文社科？
```

如果用户已在同一句话中提供其中一项，只追问缺失项：

- 已给项目名、未给路径：问“这门课走哪条路径：理工科，还是人文社科？”
- 已给路径、未给项目名：问“这次备考项目叫什么名字？”
- 两项都给出：直接初始化工作区。

## 3. 路径标准化

接受这些输入并标准化：

| 用户说法 | 标准值 |
| --- | --- |
| 理工科、理科、工科、STEM、science、engineering、计算题为主、公式推导为主 | `stem` |
| 人文社科、人文、社科、文科、liberal arts、humanities、social science、论述题为主、材料分析为主 | `humanities-social-sciences` |

如果用户的回答无法标准化，继续询问，不要进入分析。

## 4. 初始化状态字段

创建工作区时，`PROJECT.md` 和 `_analysis/exam-tutor-state.json` 都要记录路径。

状态文件至少包含：

```json
{
  "project_name": "",
  "workspace_dir": "",
  "discipline_track": "stem | humanities-social-sciences",
  "route": {
    "locked": true,
    "workflow": "references/tracks/stem/workflow.md | references/tracks/humanities-social-sciences/workflow.md",
    "selected_label": "理工科 | 人文社科"
  },
  "mode": "waiting-for-materials",
  "runtime": {
    "mode": "setup-only",
    "execution_strategy": "serial",
    "completed_phases": ["project_workspace_init"]
  }
}
```

`PROJECT.md` 的当前状态中也写明：

```markdown
- 学科路径：理工科 / 人文社科
- 路由文件：references/tracks/.../workflow.md
```

## 5. 隔离规则

路径锁定后：

1. 只能读取对应 track 下的 `workflow.md`、`output-templates.md`、`agent-team.md` 和后续子文件。
2. 不得引用另一条路径的质量标准、章节标题、教学顺序、知识点模板或 Agent 角色。
3. 公共文件只允许处理与学科无关的事情：项目目录、材料格式、状态恢复、runtime、用户确认关卡。
4. 若某条路径的内容尚未填充完整，说明缺口并生成该路径允许的最小骨架，不要临时借用另一条路径。
5. 旧项目没有 `discipline_track` 时，先询问路径并补写状态，再继续分析。

## 6. 切换路径

如果用户对已有项目说“这门课应该走另一条路径”：

- 不要直接在原目录混写新产物。
- 先说明当前项目已锁定的路径和目标路径。
- 建议创建一个新工作区，或在用户明确确认后执行迁移。
- 迁移时只复制 `materials/` 和必要的原始材料，不复制旧路径生成的 `knowledgepointslist.md`、`knowledge-points/`、`quality-review.md` 等最终产物。

## 7. 路由后的入口

路由完成后：

1. 读取当前路径的 `workflow.md`。
2. 按该 workflow 指示读取同路径的 `output-templates.md`。
3. 只有当 `runtime.execution_strategy=team` 时，才读取同路径的 `agent-team.md`；`serial`、`audit-only`、`topic-repair` 不需要读取团队分工。
4. 在状态文件中记录 `runtime.mode`、`runtime.execution_strategy`、已完成 phase、待完成输出。
5. 继续执行材料关卡、分析、知识点生成、审查或学习计划。
