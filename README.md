# exam-tutor

[English](#english) | [中文](#中文)

## English

`exam-tutor` is a reusable skill for high-yield university exam preparation.

It helps an AI agent work from real course materials, extract knowledge points, organize them by prerequisite order, ask how much time is left, and create a practical countdown study plan. It is designed for university students who need to prepare quickly and still aim for a strong result.

### Why This Skill Exists

Many university exams are driven more by the teacher's slides, textbook wording, syllabus, and past-paper patterns than by broad subject mastery.

This skill is built around four ideas:

- Materials are king. Real course materials matter more than generic explanations.
- Past papers are the most important materials, without exception. Every past-paper question must be covered by the knowledge-point system.
- Exam orientation matters. The goal is to identify likely testable points, not to cover everything.
- Application matters. Students need usable explanations and worked examples, not just summaries.

### What It Produces

When used well, the skill guides the agent to create:

- `knowledgepointslist.md`
- `study-plan.md`
- `knowledge-points/<topic>.md` files for each knowledge point

It also enforces two strict behaviors:

- every past-paper question must map to at least one knowledge point
- every knowledge-point Markdown file must analyze the related past-paper questions and connect them back to the uploaded materials

It also supports drill-down learning: if a learner gets blocked by a missing prerequisite, that prerequisite becomes a first-class knowledge point and is added back into the knowledge map and study plan.

### Expected Inputs

The skill works best when the learner provides real course materials such as:

- textbook or core readings
- PPT slides
- teacher-highlighted review scope
- syllabus or exam outline
- past papers
- class notes

Supported file types include `pdf`, `pptx`, `ppt`, `docx`, `md`, and `txt`.

### Installation

Install the skill directly from this repository:

```bash
npx skills add https://github.com/<owner>/<repo> --skill exam-tutor
```

### Recommended Companion Skill

For PDF-heavy workflows, install the `pdf` skill as a companion:

```bash
npx skills add https://github.com/openai/skills --skill pdf
```

`exam-tutor` can still operate without it, but PDF extraction and layout-sensitive review will be stronger when the `pdf` skill is available.

### Repository Structure

```text
exam-tutor/
  SKILL.md
  agents/openai.yaml
  references/
    material-processing.md
    output-templates.md
```

### Usage Example

```text
Use $exam-tutor to review these course materials, treat past papers as the most important source, build knowledgepointslist.md so every past-paper question is covered, ask how many days I have left, and create a high-yield study-plan.md.
```

### Publishing Notes

- Keep the skill course-material-driven.
- Avoid hard-coding local paths or machine-specific assumptions.
- If you publish this repo to GitHub, users can install it with the `skills` CLI and it can be indexed by skills.sh.

## 中文

`exam-tutor` 是一个面向大学生高效备考的 Skill。

它会基于真实课程资料提取知识点，按前置依赖组织知识体系，询问剩余备考时间，并生成倒计时学习计划。这个 Skill 的定位非常明确：帮助大学生在有限时间内尽可能高效地准备并争取更好的考试结果。

### 为什么要做这个 Skill

很多大学考试更接近“围绕老师 PPT、教材表述、考纲和往年题来命题”，而不是对学科深度理解的全面评估。

这个 Skill 建立在四个核心理念上：

- 材料为王：真实课程资料比通用讲解更重要。
- 往年题第一：往年题是最重要的资料，没有之一。每一道往年题都必须被知识点体系覆盖。
- 考点导向：重点是识别什么会考，而不是试图覆盖所有内容。
- 应用优先：学生需要可用于做题的解释、例题和易错点，而不只是概念摘要。

### 它会产出什么

正常使用时，这个 Skill 会引导代理生成：

- `knowledgepointslist.md`
- `study-plan.md`
- `knowledge-points/<topic>.md` 知识点文档

它还要求两个硬约束：

- 每一道往年题都必须映射到至少一个知识点
- 每个知识点文档不仅要分析相关往年题，还要深度结合上传的课程资料来解释该知识点

它也支持 Drill-down 学习：如果学生在某个知识点上卡住，发现缺少前置知识，那么这个前置知识会被提升为新的正式知识点，并同步加入知识图谱和学习计划。

### 建议输入材料

这个 Skill 最适合处理以下真实课程资料：

- 教材或核心阅读材料
- PPT 课件
- 老师划的重点或复习范围
- 考纲 / 大纲
- 往年题
- 课堂笔记

支持的主要文件格式包括 `pdf`、`pptx`、`ppt`、`docx`、`md` 和 `txt`。

### 安装方式

把这个 Skill 直接从仓库安装：

```bash
npx skills add https://github.com/<owner>/<repo> --skill exam-tutor
```

### 推荐配套 Skill

如果资料里有很多 PDF，建议同时安装 `pdf` Skill：

```bash
npx skills add https://github.com/openai/skills --skill pdf
```

即使不安装，`exam-tutor` 也能工作；但如果装了 `pdf`，PDF 提取和版面检查会更可靠。

### 仓库结构

```text
exam-tutor/
  SKILL.md
  agents/openai.yaml
  references/
    material-processing.md
    output-templates.md
```

### 使用示例

```text
Use $exam-tutor to 用中文分析这些大学课程资料，把往年题作为最重要的资料来源，构建 knowledgepointslist.md 并确保每一道往年题都被知识点覆盖，然后询问我还有多少天备考，再生成高质量的 study-plan.md。
```

### 发布说明

- 保持这个 Skill 以课程资料为中心。
- 不要写死本地路径或机器相关假设。
- 如果你把仓库发布到 GitHub，用户可以用 `skills` CLI 安装，并有机会被 skills.sh 收录。
