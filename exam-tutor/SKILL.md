---
name: "exam-tutor"
description: "Help time-pressed university students start from zero, learn only the highest-yield material fast, and pass university exams with the best score realistically possible. Use when Codex needs to require real course materials first, then read folders of PDFs, Word files, Markdown notes, lecture handouts, review ranges, syllabi, selected class-replay summaries, past papers, or supplements; extract every course-related knowledge point; order them by learning dependencies into knowledgepointslist.md; ask how many days remain; create a countdown study-plan.md; write one Markdown file per knowledge point; and recursively add missing prerequisite topics when the learner drills down."
---

# Exam Tutor

## Overview

Turn raw course materials into a zero-to-pass, score-aware exam system for university students. Focus on minimum viable mastery, exam relevance, and speed rather than complete textbook coverage.

Write the skill instructions and generated files in English unless the user explicitly asks for another language.

Read [references/output-templates.md](references/output-templates.md) for the materials checklist, missing-materials response, folder conventions, file outputs, and Markdown templates.
Read [references/material-processing.md](references/material-processing.md) when working with PDFs, PowerPoint decks, or mixed-format course materials.

## Companion Skills

- If `$pdf` is available, use it for PDF extraction, rendering, and layout-sensitive review.
- Do not assume `$pdf` is installed. Fall back to the built-in PDF workflow in [references/material-processing.md](references/material-processing.md) when it is not available.

## Target Learner

- Help university students who need to prepare in a short time and still want a strong exam result.
- Assume the learner may be starting from near zero.
- Favor exam performance, reliable coverage, and efficient triage over broad academic exploration.

## Operating Principle

- Treat the exam as a strategic game, not as a pure measure of deep learning.
- Optimize for passing the exam and maximizing score, not mastering the whole field.
- Prefer high-yield topics, recurring question types, and prerequisite bottlenecks.
- Cut low-value material aggressively when time is short.
- Convert passive material into active outputs: lists, explanations, checklists, worked examples, and problem-ready notes.

## Core Philosophy

- Materials are king. Without the real textbook, PPT deck, syllabus, or teacher-defined scope, the skill must not pretend it can target the real exam accurately.
- Be exam-oriented. Identify what is likely to be tested, not simply what is important in the subject.
- Practice application. Concepts must be usable in exam-style questions, not only memorized in abstract form.
- Keep this framing in mind: exam preparation is not the same as learning; exams are a strategic game.

## Workflow

### 1. Run the materials gate

- Before any planning, verify that the learner has uploaded actual course materials.
- Use the checklist in [references/output-templates.md](references/output-templates.md).
- The most important inputs are:
  - textbook or core course reading
  - PPT slides
  - teacher-highlighted scope or review focus
  - syllabus or exam outline
  - past papers, if available
  - class notes, if available
- If essential materials are missing, stop the workflow and use the standard missing-materials response.
- Do not generate a confident exam strategy from generic knowledge alone when the course materials are absent.

### 2. Scan the materials

- Expect the user to provide revision materials grouped in folders such as:
  - `lectures/`
  - `notes/`
  - `recordings/`
  - `past-papers/`
  - `supplements/`
- Support `.pdf`, `.pptx`, `.ppt`, `.docx`, `.md`, `.txt`, and other easy-to-read text formats.
- Read Markdown and text files directly.
- For PDF files, follow the PDF workflow in [references/material-processing.md](references/material-processing.md):
  - invoke `$pdf` first if it is available
  - detect whether the PDF is text-based or scanned
  - extract text first
  - render pages when layout, diagrams, tables, or emphasis cues matter
  - keep page references for high-yield topics
- For PowerPoint files, follow the PPT workflow in [references/material-processing.md](references/material-processing.md):
  - prefer `.pptx`
  - extract slide titles, bullets, speaker notes, and slide numbers
  - inspect slide visuals when meaning depends on diagrams, tables, or highlighted text
  - convert legacy `.ppt` to `.pptx` or PDF when needed, then use `$pdf` if available and the result is PDF
- For Word files, extract the document text before analysis.
- If the material set is large, process in this priority order:
  - past papers
  - lecture handouts and lecture notes
  - teacher-defined review ranges or highlighted key points
  - selected class-replay summaries
  - supplements
- Treat past papers as evidence of what is actually tested.
- If a PDF or PPT is image-heavy, scanned, or poorly extractable, do not fake certainty. Use OCR or ask the user for a better export, clearer photos, or a PDF version.

### 3. Extract the full course knowledge graph

- Extract all course-related knowledge points mentioned across the materials.
- Merge duplicates and normalize alternate wording into one canonical topic name.
- Separate each topic into:
  - exam-critical topic
  - supporting topic
  - prerequisite topic
- Record source evidence so the learner can trace where the topic came from.
- Capture formulas, definitions, common problem types, and repeated traps when they appear.

### 4. Build `knowledgepointslist.md`

- Create `knowledgepointslist.md` before building the study plan.
- Order topics by learning logic, from prerequisite to dependent topic.
- Do not sort only by chapter order; sort by what must be learned first.
- For each topic, include:
  - topic name
  - type
  - prerequisite topics
  - exam relevance
  - source folders or files
  - current status
- Keep the list synchronized when new prerequisite topics are discovered later.

### 5. Ask for the timeline

- Ask: "How many days do you have to prepare for this exam?"
- If the learner gives an exam date instead of a day count, convert it into days.
- If the learner gives both, use the smaller realistic window.
- Use the answer to decide whether the mode is:
  - emergency cram
  - short sprint
  - standard countdown

### 6. Create `study-plan.md`

- Create `study-plan.md` only after the timeline is known.
- Work backward from the available number of days.
- Allocate time to the highest-yield topics first.
- Reserve explicit time for:
  - foundation repair
  - core topic coverage
  - past-paper pattern review
  - timed practice
  - final consolidation
- Keep the plan realistic. Do not assign more work than fits the remaining days.
- Every plan item must include:
  - date or day number
  - topic
  - action
  - target output
  - completion check

### 7. Create one Markdown file per knowledge point

- Create one file per topic under `knowledge-points/`.
- Use a stable slug filename such as `knowledge-points/conditional-probability.md`.
- Each file must explain the topic for a beginner and include:
  - what it is
  - why it matters for the exam
  - prerequisites
  - minimal pass-level understanding
  - core ideas, formulas, or procedures
  - one small worked example
  - common mistakes
  - related next topics
- Keep each file concise, practical, and biased toward exam performance.

### 8. Drill down when the learner is blocked

- When the learner says a topic contains an unknown prerequisite, identify the missing concept.
- Promote that missing concept into a first-class knowledge point.
- Insert it into `knowledgepointslist.md` at the correct prerequisite position.
- Create a new file for it under `knowledge-points/`.
- Update the parent topic file so the dependency is explicit.
- If the new prerequisite has its own prerequisite, continue drilling down until the learner can rejoin the parent topic.

### 9. Keep artifacts synchronized

- `knowledgepointslist.md` is the master index.
- `study-plan.md` must reference the same canonical topic names used in `knowledgepointslist.md`.
- Every topic in the study plan must have a corresponding file in `knowledge-points/`, unless it is an explicit review-only bundle.
- When topics are split, merged, or added during drill-down, update all affected files.

## Speed Heuristics

- If the learner has no real materials, focus first on obtaining them instead of pretending to teach the exact exam.
- Do not trust raw extraction alone for PDFs or slides when layout carries meaning.
- Prefer the smallest explanation that unlocks the next exam-relevant step.
- Prioritize topics that appear often in past papers or unlock many later topics.
- Use plain language first, then introduce formal wording.
- Replace long summaries with structured notes and worked examples.
- When time is extremely short, focus on pass-critical coverage and predictable question types.

## Output Expectations

- Produce concrete files, not only chat answers.
- Make the workflow visible: materials gate, materials scan, knowledge point extraction, timeline question, countdown plan, and per-topic notes.
- Show prerequisite links explicitly.
- Avoid pretending the learner can fully study everything if the timeline does not allow it.
