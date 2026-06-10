# exam-tutor

[中文](#中文) | [English](#english)

## 中文

`exam-tutor` 是一个兼容 Claude Code、Codex、Kimi Code 的 Agent Skill。它把一门课的真实资料、讲义、PPT、笔记、历年真题转化成一个以考试得分为目标的备考工作区。新项目会先锁定学科路径：理工科或人文社科；两条路径使用彼此隔离的 workflow、模板和质量标准。

### 有什么用

给一门课的真实材料后，`exam-tutor` 会帮你：

1. **逐字提取历年真题**，建立稳定的 `_analysis/question-index.json` 真题索引。
2. **构建知识图谱**，生成 `knowledgepointslist.md`，确保每道真题都映射到至少一个知识点。
3. **为每个知识点生成教学文件**，放在 `knowledge-points/` 下。每个文件包含：
   - 对应学科路径的核心讲解
   - 材料证据和考试用途
   - 理工科题型流程或人文社科答题框架
   - 相关真题、样题或材料重点解析
4. **做覆盖审计和质量检查**，避免漏题、错题、题目 ID 不一致。
5. **根据剩余天数生成学习计划**，输出 `study-plan.md`。

核心目标不是“学完整本教材”，而是在有限时间内建立可执行、可检查、以真题为中心的备考体系。

### 一句话使用

在你希望存放备考项目的目录中打开 Claude Code、Codex 或 Kimi Code，然后直接告诉 agent：

```text
下载 https://github.com/skywalkjian/exam-tutor-skill，并使用 exam-tutor 开始一个新的备考项目。
```

预期流程：

1. agent 下载或安装 `skywalkjian/exam-tutor-skill`。
2. agent 询问：“这次备考项目叫什么名字？这门课属于哪条路径：理工科，还是人文社科？”
3. 你回答项目名和学科路径。
4. agent 用这个名字创建新的工作区，并在状态文件中锁定路径。
5. agent 在工作区里创建 `materials/`、`_analysis/`、`knowledge-points/` 等目录。
6. agent 停下来，等待你放入课程资料。
7. 你把资料放进 `materials/` 后，说“开始工作”。
8. agent 开始提取真题、构建知识图谱、生成知识点文件和学习计划。

### 一键安装

如果你的 agent 不能自动安装 skill，可以手动运行下面的命令。它会把同一个 `exam-tutor/` skill 文件夹安装到 Claude Code、Codex、Kimi Code 的用户级 skills 目录：

```bash
tmp="$(mktemp -d)" && \
git clone --depth 1 https://github.com/skywalkjian/exam-tutor-skill.git "$tmp/exam-tutor-skill" && \
for dir in "$HOME/.claude/skills" "${CODEX_HOME:-$HOME/.codex}/skills" "${KIMI_CODE_HOME:-$HOME/.kimi-code}/skills"; do
  mkdir -p "$dir"
  rm -rf "$dir/exam-tutor"
  cp -R "$tmp/exam-tutor-skill/exam-tutor" "$dir/exam-tutor"
done && \
rm -rf "$tmp" && \
echo "Installed exam-tutor for Claude Code, Codex, and Kimi Code. Restart the tool you use."
```

如果希望把 skill 跟某门课的资料仓库一起提交，也可以项目级安装：

```bash
for dir in .claude/skills .codex/skills .agents/skills .kimi-code/skills; do
  mkdir -p "$dir"
  rm -rf "$dir/exam-tutor"
  cp -R /path/to/exam-tutor-skill/exam-tutor "$dir/exam-tutor"
done
```

### 具体使用方法

安装后重启对应工具，在你希望创建课程工作区的父目录启动会话，然后按工具选择调用方式：

| 工具 | 调用方式 |
|---|---|
| Claude Code | `请使用 exam-tutor 帮我准备考试` |
| Codex CLI / IDE / app | `$exam-tutor 帮我准备考试` |
| Kimi Code | `skill: exam-tutor 帮我准备考试` |

更明确的提示词：

```text
开始一个新的 exam-tutor 备考项目。
```

agent 会先创建项目工作区。你随后把课程资料放入该工作区的 `materials/`：

```text
materials/
  lectures/      ← 教材、讲义、课件 PDF
  notes/         ← 课堂笔记、个人笔记
  recordings/    ← 课堂录音摘要
  past-papers/   ← 历年真题（最重要）
  supplements/   ← 补充材料、习题集
```

支持格式：`pdf`、`pptx`、`ppt`、`docx`、`md`、`txt`。

放好资料后告诉 agent：

```text
开始工作
```

如果资料包含 PDF，建议在你使用的 agent 环境中启用 PDF 读取 skill 或工具。没有 PDF 工具也能运行，但提取质量会取决于宿主工具对 PDF 的读取能力。

### 学科路径

`exam-tutor` 现在有两条隔离路径：

| 路径 | 适合课程 | 核心产出风格 |
|---|---|---|
| 理工科 | 数学、物理、计算机、工程、统计、偏公式推导和计算题的课程 | 从零直觉、定义/符号、公式推导、例题、题型流程、真题解析 |
| 人文社科 | 文史哲、政治学、社会学、法学、传播、偏概念论述和材料分析的课程 | 问题意识、概念界定、理论谱系、争议结构、案例材料、答题框架 |

路径写入 `_analysis/exam-tutor-state.json` 后会被锁定。后续分析只读取对应 `references/tracks/<track>/` 下的内容。

### 产出文件

```text
knowledgepointslist.md          ← 知识图谱 + Mermaid 依赖关系图
study-plan.md                   ← 倒计时学习计划
knowledge-points/
  KP-01-xxx.md                  ← 按学科路径生成的专题讲义 + 题目/答题训练
  KP-02-xxx.md
  ...
_analysis/
  exam-tutor-state.json         ← 可恢复执行状态
  question-index.json           ← 每道真题的稳定索引
  question-index.md             ← 人类可读版真题索引
  coverage-audit.md             ← 逐题覆盖审计
  quality-review.md             ← 质量审查报告
```

### 原理和结构

`exam-tutor` 的核心原则：

- **材料为王**：没有真实课程资料，不生成看似自信的通用备考策略。
- **真题第一**：历年真题是最高优先级证据，每道题都必须覆盖。
- **先索引，后生成**：先建立 `_analysis/question-index.json`，后续所有文件都引用同一批题目 ID。
- **可恢复执行**：状态写入 `_analysis/exam-tutor-state.json`，中断后可以继续。
- **最小必要更新**：新增材料或修补某个知识点时，不默认重写全部文件。
- **路径隔离**：理工科和人文社科分别使用独立 workflow、输出模板和 Agent 分工。
- **从零教起**：假设学习者基础薄弱，按对应学科路径建立可考试输出的理解。

runtime 规则拆分在 `exam-tutor/references/runtime/`：

| 文件 | 作用 |
|---|---|
| `index.md` | runtime 入口索引 |
| `_detect.md` | 检测宿主工具、用户意图、runtime mode 和 execution strategy |
| `phase-barrier.md` | 阶段屏障、resume、增量更新 |
| `create-agent.md` | 统一 Agent 创建契约 |
| `claude-team.md` | Claude Code 适配 |
| `codex-subagent.md` | Codex 适配 |
| `kimi-team.md` | Kimi Code 适配 |

材料较多或跨类别且宿主确认支持并行 Agent 时，skill 会使用分阶段并行 Agent 工作流；材料较少、宿主不支持或无法确认并行能力时，会使用 `serial` fallback，但关键产物保持一致。

学科路径内容拆分在 `exam-tutor/references/tracks/`：

| 文件夹 | 作用 |
|---|---|
| `stem/` | 理工科 workflow、输出模板、Agent 分工 |
| `humanities-social-sciences/` | 人文社科 workflow、输出模板、Agent 分工骨架 |

### 环境要求

- Claude Code、Codex、Kimi Code 三者之一
- `git`，用于下载或安装 skill
- Python + `numpy` + `sympy`，仅理工科路径中的繁重数学计算需要

### 项目状态

**2026-04-26**

1. 目前作者本人只在一门广度大、深度浅的纯数学课程上进行过使用，数据有限，多有不足。希望广泛吸收各种课程的使用测评和反馈，用于进一步改进。
2. 本项目长期维护，长期欢迎合作者。
3. 有考虑建立工具库，该方面目前尚未开始建设。

---

## English

`exam-tutor` is an Agent Skill for Claude Code, Codex, and Kimi Code. It turns real course materials, lecture notes, slides, and past papers into a score-oriented exam prep workspace. New projects first choose and lock a discipline track: STEM or humanities/social sciences.

### What It Does

Given real course materials, `exam-tutor` will:

1. Extract every past-paper question verbatim into `_analysis/question-index.json`.
2. Build a knowledge map in `knowledgepointslist.md`.
3. Generate one teaching file per topic under `knowledge-points/`.
4. Use the locked track’s teaching style: STEM problem workflows or humanities/social-sciences answer frameworks.
5. Run coverage review and create a countdown `study-plan.md`.

### One-Line Use

Open Claude Code, Codex, or Kimi Code in the folder where you want exam projects to live, then say:

```text
Download https://github.com/skywalkjian/exam-tutor-skill and use exam-tutor to start a new exam prep project.
```

Expected flow:

1. The agent downloads or installs the skill.
2. The agent asks for the project name and discipline track.
3. You provide both.
4. The agent creates a dedicated workspace and locks the route in `_analysis/exam-tutor-state.json`.
5. You add materials under `materials/`.
6. You say “start working”.
7. The agent starts the full exam-tutor workflow.

### Manual Install

```bash
tmp="$(mktemp -d)" && \
git clone --depth 1 https://github.com/skywalkjian/exam-tutor-skill.git "$tmp/exam-tutor-skill" && \
for dir in "$HOME/.claude/skills" "${CODEX_HOME:-$HOME/.codex}/skills" "${KIMI_CODE_HOME:-$HOME/.kimi-code}/skills"; do
  mkdir -p "$dir"
  rm -rf "$dir/exam-tutor"
  cp -R "$tmp/exam-tutor-skill/exam-tutor" "$dir/exam-tutor"
done && \
rm -rf "$tmp" && \
echo "Installed exam-tutor for Claude Code, Codex, and Kimi Code. Restart the tool you use."
```

### Detailed Usage

After installing, restart your agent and invoke the skill:

| Tool | Invocation |
|---|---|
| Claude Code | `请使用 exam-tutor 帮我准备考试` |
| Codex CLI / IDE / app | `$exam-tutor 帮我准备考试` |
| Kimi Code | `skill: exam-tutor 帮我准备考试` |

Put materials into the created workspace:

```text
materials/
  lectures/
  notes/
  recordings/
  past-papers/
  supplements/
```

Then tell the agent:

```text
start working
```

### Outputs and Internals

Key outputs:

```text
knowledgepointslist.md
study-plan.md
knowledge-points/
_analysis/question-index.json
_analysis/coverage-audit.md
_analysis/exam-tutor-state.json
```

Runtime rules live under `exam-tutor/references/runtime/`. The workflow first creates a project workspace and waits for materials; after you say “start working”, it builds the question index, knowledge map, topic files, review reports, and study plan in dependency order.

Track-specific content lives under `exam-tutor/references/tracks/`:

- `stem/`: STEM workflow, output templates, and agent roles.
- `humanities-social-sciences/`: humanities/social-sciences workflow, output templates, and agent-role skeleton.
