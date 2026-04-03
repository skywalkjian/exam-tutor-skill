# exam-tutor

`exam-tutor` is a reusable skill for high-yield university exam preparation.

It helps an AI agent work from real course materials, extract knowledge points, organize them by prerequisite order, ask how much time is left, and create a practical countdown study plan. It is designed for university students who need to prepare quickly and still aim for a strong result.

## Why This Skill Exists

Many university exams are driven more by the teacher's slides, textbook wording, syllabus, and past-paper patterns than by broad subject mastery.

This skill is built around three ideas:

- Materials are king. Real course materials matter more than generic explanations.
- Past papers are the most important materials, without exception. Every past-paper question must be covered by the knowledge-point system.
- Exam orientation matters. The goal is to identify likely testable points, not to cover everything.
- Application matters. Students need usable explanations and worked examples, not just summaries.

## What It Produces

When used well, the skill guides the agent to create:

- `knowledgepointslist.md`
- `study-plan.md`
- `knowledge-points/<topic>.md` files for each knowledge point

It also requires two strict behaviors:

- every past-paper question must map to at least one knowledge point
- every knowledge-point Markdown file must analyze the related past-paper questions, not just explain the concept in isolation

It also supports drill-down learning: if a learner gets blocked by a missing prerequisite, that prerequisite becomes a first-class knowledge point and is added back into the knowledge map and study plan.

## Expected Inputs

The skill works best when the learner provides real course materials such as:

- textbook or core readings
- PPT slides
- teacher-highlighted review scope
- syllabus or exam outline
- past papers
- class notes

Supported file types include `pdf`, `pptx`, `ppt`, `docx`, `md`, and `txt`.

## Installation

Install the skill directly from this repository:

```bash
npx skills add https://github.com/<owner>/<repo> --skill exam-tutor
```

The `skills` CLI install pattern is documented on the Skills docs site.

## Recommended Companion Skill

For PDF-heavy workflows, install the `pdf` skill as a companion:

```bash
npx skills add https://github.com/openai/skills --skill pdf
```

`exam-tutor` can still operate without it, but PDF extraction and layout-sensitive review will be stronger when the `pdf` skill is available.

## Repository Structure

```text
exam-tutor/
  SKILL.md
  agents/openai.yaml
  references/
    material-processing.md
    output-templates.md
```

## Usage Example

```text
Use $exam-tutor to review these course materials, treat past papers as the most important source, build knowledgepointslist.md so every past-paper question is covered, ask how many days I have left, and create a high-yield study-plan.md.
```

## Publishing Notes

- Keep the skill generic and course-material-driven.
- Avoid hard-coding local paths or machine-specific assumptions.
- If you publish this repo to GitHub, users can install it directly with the `skills` CLI and it can be indexed by skills.sh.
