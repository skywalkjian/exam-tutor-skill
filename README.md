# exam-tutor

**2026-04-26**
对项目后续的期待

1 目前作者本人只在一门广度大深度浅的纯数学课程上进行过使用，数据有限，多有不足。希望能够广泛吸收各种课程的使用测评和使用反馈，用于进一步改进。包括但不限于：不同特性的课程的针对性优化，不同院系课程的针对性优化，不同考试要求的课程的针对性优化
2 本项目长期维护，长期欢迎合作者。
3 有考虑建立工具库，该方面目前尚未开始建设

---
[English](#english) | [中文](#中文)

## English

`exam-tutor` is a Claude Code skill that turns raw course materials into a complete, score-oriented exam prep system for university students.

### What It Does

Given a folder of real course materials, the skill:

1. **Scans and extracts** every past-paper question verbatim, then analyzes lectures, slides, notes, and supplements
2. **Builds a knowledge map** (`knowledgepointslist.md`) with a Mermaid dependency graph, ensuring every past-paper question maps to at least one knowledge point
3. **Generates a knowledge-point file for each topic** (`knowledge-points/KP-01-topic.md`) — each file is a self-contained mini-lesson that includes:
   - Intuition-first teaching (problem → intuition → concrete example → formal derivation → worked example)
   - **General problem-solving workflows** for each question type under this topic
   - **Full solutions to every related past-paper question**, with strategy before step-by-step solving
4. **Asks how many days you have left** and creates a realistic countdown study plan (`study-plan.md`)

### Core Ideas

- **Materials are king.** The skill refuses to generate exam strategy without real course materials.
- **Past papers come first.** They are the strongest evidence of what will be tested. Every question must be covered.
- **Knowledge points exist to serve problem-solving.** Each topic file includes generalized solving workflows and full past-paper solutions, not just concept summaries.
- **Teach from zero.** Assumes the learner may be starting from scratch; uses plain-language intuition and everyday analogies before formal definitions.

### What It Produces

```
knowledgepointslist.md          ← Knowledge map with Mermaid dependency graph
study-plan.md                   ← Countdown study plan based on available days
knowledge-points/
  KP-01-topic-one.md            ← Full teaching + problem-type workflows + past-paper solutions
  KP-02-topic-two.md
  ...
_analysis/                      ← Intermediate analysis files (agent team outputs)
```

### Expected Inputs

Place your course materials in a `materials/` folder (the skill will create this structure if it doesn't exist):

```
materials/
  lectures/      ← Textbooks, lecture handouts, course PDFs
  notes/         ← Class notes, personal notes
  recordings/    ← Class recording summaries
  past-papers/   ← Past exam papers (most important!)
  supplements/   ← Supplementary materials, exercise sets
```

Supported formats: `pdf`, `pptx`, `ppt`, `docx`, `md`, `txt`.

### Installation

```bash
claude skill add --from https://github.com/skywalkjian/exam-tutor-skill
```

### Recommended: Install the PDF Skill

If your materials contain PDFs (most do), install the `pdf` skill for better extraction:

```bash
claude skill add --from https://github.com/anthropics/claude-code-pdf-skill
```

The skill works without it, but PDF extraction quality improves significantly with `$pdf` available.

### Usage

Start a Claude Code session in the directory containing your materials, then:

```
$exam-tutor 帮我准备考试
```

Or be more specific:

```
$exam-tutor 分析 materials/ 文件夹里的课程资料，构建知识体系，生成知识点文件和学习计划
```

The skill will:
1. Check that you have real course materials (and tell you what's missing)
2. Scan and analyze everything, prioritizing past papers
3. Build the knowledge map and generate all knowledge-point files
4. Ask you how many days you have left
5. Create a study plan tailored to your timeline

### How the Agent Team Works

When materials contain 3+ files, the skill automatically uses parallel agents:

- **Wave 1**: One agent per past paper + one for slides/notes + one for supplements (all parallel)
- **Wave 2**: One agent per lecture file, guided by past-paper high-frequency topics (all parallel)
- **Knowledge map**: Main process merges all analysis results
- **Knowledge-point files**: One writing agent per topic (all parallel), each producing a full teaching document with solutions
- **Quality review**: A review agent checks coverage, consistency, and depth
- **Final**: Corrections applied, study plan generated

### Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Python with `numpy` and `sympy` (for heavy math computation in solutions)

---

## 中文

`exam-tutor` 是一个 Claude Code 技能，能将原始课程资料转化为完整的、以分数为导向的大学考试备考体系。

### 它做什么

给定一个包含真实课程资料的文件夹，这个技能会：

1. **扫描并逐字提取**每一道往年真题，然后分析讲义、PPT、笔记和补充材料
2. **构建知识图谱**（`knowledgepointslist.md`），含 Mermaid 依赖关系图，确保每道真题都映射到至少一个知识点
3. **为每个知识点生成一个教学文件**（`knowledge-points/KP-01-topic.md`）——每个文件是一堂完整的迷你课，包含：
   - 直觉优先的教学流程（问题 → 直觉 → 具体案例 → 严格推导 → 完整例题）
   - **该知识点下每种题型的通用解题流程**（步骤化模板，遇到同类题直接套用）
   - **该知识点下每道真题的完整解析**（先给整体解题思路，再逐步解题）
4. **询问你还有多少天**，生成现实可行的倒计时学习计划（`study-plan.md`）

### 核心理念

- **材料为王**：没有真实课程资料，技能会拒绝生成考试策略
- **真题第一**：往年题是考试考什么、怎么考的最强证据，每道题必须被覆盖
- **知识点为做题服务**：每个知识点文件不仅教概念，还包含题型通用解题流程和完整真题解析
- **从零教起**：假设学习者可能零基础，用大白话和生活化类比建立直觉，再引入公式

### 产出文件

```
knowledgepointslist.md          ← 知识图谱 + Mermaid 依赖关系图
study-plan.md                   ← 倒计时学习计划
knowledge-points/
  KP-01-xxx.md                  ← 完整教学 + 题型解题流程 + 真题解析
  KP-02-xxx.md
  ...
_analysis/                      ← Agent 团队中间分析结果
```

### 输入材料

将课程资料放入 `materials/` 文件夹（技能会自动创建目录结构）：

```
materials/
  lectures/      ← 教材、讲义、课件 PDF
  notes/         ← 课堂笔记、个人笔记
  recordings/    ← 课堂录音摘要
  past-papers/   ← 历年真题（最重要！）
  supplements/   ← 补充材料、习题集
```

支持格式：`pdf`、`pptx`、`ppt`、`docx`、`md`、`txt`。

### 安装

```bash
claude skill add --from https://github.com/skywalkjian/exam-tutor-skill
```

### 推荐：安装 PDF 技能

如果资料包含 PDF（大多数情况都有），建议安装 `pdf` 技能以获得更好的提取效果：

```bash
claude skill add --from https://github.com/anthropics/claude-code-pdf-skill
```

不安装也能用，但有 `$pdf` 时 PDF 提取质量会显著提升。

### 使用方法

在包含课程资料的目录下启动 Claude Code，然后：

```
$exam-tutor 帮我准备考试
```

或者更具体地：

```
$exam-tutor 分析 materials/ 文件夹里的课程资料，构建知识体系，生成知识点文件和学习计划
```

技能会自动：
1. 检查你是否有真实课程资料（缺什么会告诉你）
2. 扫描分析所有材料，优先处理往年真题
3. 构建知识图谱，生成所有知识点文件（含题型流程和真题解析）
4. 问你还有几天考试
5. 根据时间线生成学习计划

### Agent 团队机制

当材料超过 3 个文件时，技能自动启用并行 Agent 团队：

- **第一波**：每份真题一个 Agent + 课件笔记一个 Agent + 补充材料一个 Agent（并行）
- **第二波**：每个讲义文件一个 Agent，由真题高频考点引导分析优先级（并行）
- **知识图谱**：主流程合并所有分析结果
- **知识点文件**：每个知识点一个撰写 Agent（并行），每个产出完整教学文档 + 解题流程 + 真题解析
- **质量审查**：审查 Agent 检查覆盖、一致性和深度
- **最终**：修正问题，生成学习计划

### 环境要求

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Python + `numpy` + `sympy`（用于解题中的数学计算）
