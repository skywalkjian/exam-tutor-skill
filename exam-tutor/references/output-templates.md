# Output Templates

Use these templates to create the concrete files and gating responses that this skill should maintain.

## Materials Gate

Use this checklist before any planning or knowledge extraction:

```markdown
## Materials Checklist
- [ ] Textbook or course reading
- [ ] PPT slides
- [ ] Teacher-highlighted key points or review scope
- [ ] Syllabus or exam outline
- [ ] Past papers, if available
- [ ] Class notes, if available
```

## Standard Missing-Materials Response

Use this response when the learner wants exam prep help but has not uploaded real course materials yet:

```markdown
I understand that you want to prepare for the exam, but there is one critical issue first: I need your textbook, PPT slides, or syllabus to help you effectively.

University exams are usually based on what the teacher taught in class and on the specific knowledge points shown in the textbook or slides. If I help only from generic knowledge, three problems appear immediately:

- My wording may not match the teacher's wording.
- The topics I emphasize may not be the topics your teacher will test.
- The examples I choose may be different from the textbook and exam style.

Please upload your course materials first. Even PPT slides, photos of the review scope, or a syllabus screenshot are enough to start.
```

## Expected Input Layout

Prefer a folder structure like this when the user provides materials:

```text
materials/
  lectures/
  notes/
  recordings/
  past-papers/
  supplements/
```

The actual folder names can vary. Map them to the nearest category and continue.

## Expected Output Layout

```text
knowledgepointslist.md
study-plan.md
knowledge-points/
  topic-one.md
  topic-two.md
```

## 1. `knowledgepointslist.md`

```markdown
# Knowledge Points List

## Course
- Name:
- Materials scanned:
- Current planning mode: Emergency Cram / Short Sprint / Standard Countdown

## Past-Paper Coverage Summary
- Total past-paper questions found:
- Covered by knowledge points:
- Uncovered questions:
- Coverage status: Pass / Incomplete

## Ordered Knowledge Points
| Reading Order | Topic | Type | Prerequisites | Why Learn It Now | Exam Relevance | Linked Past-Paper Questions | Linked Material Evidence | Main Sources | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| KP-01 | Conditional Probability | Prerequisite | Basic probability, event notation | Unlocks Bayes theorem and word problems | High | 2024 Midterm Q2; 2023 Final Q1(b) | lecture-03.pdf p.12; week-3-slides.pdf slide 8; notes/probability.md | lectures/week-3.pdf; past-papers/2024-midterm.pdf | Not started |
| KP-02 | Bayes Theorem | Exam-critical | Conditional Probability | Frequent exam target | High | 2023 Final Q4; 2022 Resit Q3 | slides/bayes.pdf slide 14; notes/bayes.md; recordings/week-5-summary.md | notes/bayes.md; past-papers/finals-2023.pdf | Not started |

## Uncovered Past-Paper Questions
- None
- If not none, list every uncovered question explicitly and do not claim the knowledge graph is complete.

## Notes
- Keep the order dependency-first, not chapter-first.
- Do not allow any past-paper question to remain uncovered.
- Insert newly discovered prerequisite topics into the correct place instead of appending them at the end.
```

## 2. `study-plan.md`

```markdown
# Study Plan

## Timeline
- Total days available:
- Planning mode:
- Main risk:
- Passing strategy:

## Phase Plan
### Phase 1: Foundation Repair
- Days:
- Topics:
- Exit criteria:

### Phase 2: High-Yield Core Topics
- Days:
- Topics:
- Exit criteria:

### Phase 3: Past-Paper Pattern Training
- Days:
- Topics:
- Exit criteria:

### Phase 4: Final Review
- Days:
- Topics:
- Exit criteria:

## Daily Schedule
| Day | Topics | Tasks | Required Output | Completion Check |
| --- | --- | --- | --- | --- |
| Day 1 | KP-01 Conditional Probability | Read topic file, solve 8 basic questions, explain the definition aloud | Updated notes + solved set | Can solve 6/8 without notes |

## Non-Negotiables
- Review blocks:
- Practice blocks:
- Buffer:
- Final mock:
```

## 3. `knowledge-points/<slug>.md`

```markdown
# KP-01 Topic: Conditional Probability

## Role in the Course
- Type: Prerequisite / Supporting / Exam-critical
- Reading order:
- Why it matters:
- Where it appears in the materials:
- Linked past-paper questions:

## Material Evidence
- Lecture or textbook source:
- PPT or slide source:
- Notes source:
- Recording or verbal source:
- Supplement source:
- Teacher wording worth preserving:

## Learning Goal
- After reading this file, the learner should be able to:
- Why this topic is hard for beginners:

## What You Must Understand to Pass
- Minimum pass-level understanding:
- What can be ignored for now:

## Prerequisites
- Required first:
- Optional supporting ideas:

## Core Teaching Section
- This is the most important and longest part of the file.
- Spend most of the writing effort here.
- Keep the logic smooth and easy to follow.

## 1. The Problem
- Start with an engineering dilemma, physical paradox, or practical failure case:
- Why this problem matters:
- What goes wrong if the learner does not understand this topic:

## 2. The Intuition
- Give a daily-life analogy:
- Explain the abstract idea through physical intuition:
- Which material explains this intuition best:

## 3. Concrete STEM Case
- Use a real engineering, physical, or STEM scenario:
- Input:
- Output:
- What changes in the middle:
- How this connects to the real concept:

## 4. The Rigor
- Formal definition or formula:
- Explain every symbol:
- Explain the physical or logical meaning of each symbol:
- Step-by-step derivation or logic breakdown:
- When this formula or method works:
- When this formula or method does not work:
- How the lecture or textbook presents it:
- How the PPT frames or compresses it:
- How notes or recordings clarify it:
- Avoid unnecessary new terminology:
- If a new term must appear, explain it immediately in plain Chinese:

## 5. Worked Example
- Problem:
- Full solution:
- Why the solution works:
- Why each step is necessary:

## Past-Paper Analysis
- If relevant, copy or restate the past-paper question first:
- Question reference:
- Full question text or essential content:
- How this topic is tested in that question:
- Fast recognition pattern:
- Detailed solution and reasoning:
- Solving steps that earn marks:
- Common trap shown by this question:

## Material-to-Exam Bridge
- Which material most directly prepares the learner for this past-paper question:
- What the learner should notice in the material before attempting the question:
- Which exact phrase, formula, diagram, or example from the material reappears in the exam setting:

## Common Mistakes
- Mistake 1:
- Mistake 2:
- How to avoid these mistakes:

## Quick Self-Check
- Question 1:
- Question 2:
- Question 3:

## Mastery Checklist
- [ ] I can explain this topic in plain language.
- [ ] I know when to use it.
- [ ] I know when not to use it.
- [ ] I can solve a standard exam question on it.
- [ ] I can spot the common trap.

## Links
- Parent topics unlocked by this topic:
- Next topics to study:
```

## 4. Drill-Down Update Pattern

Use this whenever the learner says a topic still depends on something unknown.

```markdown
## Drill-Down Record
- Parent topic:
- Missing prerequisite discovered:
- Why it blocks progress:
- New topic file created:
- `knowledgepointslist.md` updated: Yes / No
- `study-plan.md` updated: Yes / No
- Parent topic file updated: Yes / No
- Past-paper coverage rechecked: Yes / No
```

## Timeline Rules

- Use `Emergency Cram` when the learner has 7 days or fewer.
- Use `Short Sprint` when the learner has 8 to 21 days.
- Use `Standard Countdown` when the learner has more than 21 days.
