# 输出模板路由

本文件是兼容入口，不再提供具体学科路径的最终模板。

使用 `exam-tutor` 时，先按 [domain-router.md](domain-router.md) 确定并锁定 `discipline_track`，然后只读取对应路径的模板：

- 理工科：`references/tracks/stem/output-templates.md`
- 人文社科：`references/tracks/humanities-social-sciences/output-templates.md`

## 公共约束

- 未锁定学科路径前，不生成 `knowledgepointslist.md`、`knowledge-points/` 或 `study-plan.md`。
- 锁定路径后，不要从另一条路径复制标题、质量标准或输出结构。
- 公共层只负责项目目录、材料目录、状态文件和 runtime 信息；学科内容由 track 模板定义。

## 公共项目目录

两个路径共享同一外层目录约定：

```text
<project-name>/
  PROJECT.md
  materials/
    lectures/
    notes/
    recordings/
    past-papers/
    supplements/
  _analysis/
    questions/
    exam-tutor-state.json
  knowledge-points/
```

初始化状态必须包含：

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
