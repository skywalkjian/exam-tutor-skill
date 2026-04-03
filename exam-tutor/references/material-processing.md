# Material Processing

Use this reference when the learner provides course materials in PDF, PowerPoint, Word, Markdown, or mixed folders.

## Goal

Convert each source into reliable exam evidence without losing teacher emphasis, page context, or diagram meaning.

## General Rules

- Prefer the original course material over summaries.
- When past papers exist, extract and index them first.
- Keep source references such as page numbers, slide numbers, file names, and section headings.
- When a source is PDF and `$pdf` is available, invoke `$pdf` before doing exam-specific analysis.
- Do not rely on text extraction alone when layout, color emphasis, tables, formulas, or diagrams carry meaning.
- If extraction quality is poor, say so clearly and ask for a better export instead of pretending confidence.

## PDF Workflow

### When to use

- Textbook chapters
- Exported PPT slide decks saved as PDF
- Syllabus files
- Past papers
- Scanned handouts

### Steps

1. If `$pdf` is available, invoke `$pdf` and follow its extraction and rendering workflow.
2. Detect the PDF type.
   - If selectable text exists, treat it as text-based.
   - If pages are image-only, treat it as scanned.
3. Extract text first for fast indexing.
4. Render pages when any of these matter:
   - tables
   - formulas
   - diagrams
   - highlighted teacher emphasis
   - strange extraction results
5. Preserve source anchors:
   - file name
   - page number
   - chapter or section title
6. Mark confidence:
   - `high` for clean text-based extraction
   - `medium` for mixed extraction needing visual confirmation
   - `low` for scanned or damaged PDFs

### PDF-specific heuristics

- For past papers, prioritize question stems, repeated patterns, mark allocations, and solution structure.
- For past papers, also preserve question identifiers such as year, term, section, question number, and sub-question labels.
- For past papers, extract enough detail to map each question to one or more knowledge points later.
- For textbooks, prioritize definitions, theorem statements, worked examples, summary boxes, and end-of-chapter exercises.
- For teacher-made handouts, prioritize bolded text, repeated warning notes, and anything labeled review, summary, key point, or example.

### If the PDF is scanned

- Prefer OCR if available.
- If OCR is unavailable or poor, ask the user for:
  - a clearer PDF
  - higher-resolution photos
  - the original digital file
  - a teacher-exported PDF

## PowerPoint Workflow

### Supported formats

- Prefer `.pptx`
- Accept `.ppt`, but treat it as legacy and convert it first when practical

### Steps

1. Identify the deck type.
   - text-heavy lecture slides
   - diagram-heavy explanation slides
   - summary or review slides
   - problem-solving slides
2. For `.pptx`:
   - extract slide titles
   - extract bullet points
   - extract speaker notes if they exist
   - preserve slide numbers
3. For `.ppt`:
   - convert to `.pptx` or PDF first when possible
   - if conversion is not possible, ask the user to export it
   - if converted to PDF and `$pdf` is available, invoke `$pdf`
4. Inspect visuals whenever the slide meaning depends on:
   - diagrams
   - charts
   - tables
   - equation layout
   - colored or animated emphasis
5. Treat each slide as exam evidence, not just presentation content.

### PPT-specific heuristics

- Prioritize:
  - title slides that define the unit
  - summary slides
  - slides with example problems
  - slides with formulas or derivations
  - slides with highlighted or colored text
  - slides repeated in class notes or past-paper topics
- If a slide is sparse, inspect neighboring slides before concluding it is low value.
- If speaker notes exist, treat them as high-value teacher intent.

## Word and Markdown Workflow

- Read `.docx`, `.md`, and `.txt` directly when possible.
- Preserve headings and list structure.
- Keep explicit teacher wording when it signals likely exam phrasing.

## What to Save During Extraction

For each high-yield point, keep:

- canonical topic name
- source file
- page or slide number
- source quote or paraphrase
- whether it looks definitional, procedural, or exam-applied
- extraction confidence

For each past-paper question, also keep:

- year and exam name
- question number
- sub-question labels if present
- tested skill or likely knowledge point
- marks or weight if visible
- any recurring trap or solving pattern

## Failure Cases

Stop and ask for better input if:

- the PDF is unreadable
- the PPT is legacy `.ppt` and cannot be converted
- the material is mostly photos with missing pages
- formulas are broken by extraction
- diagrams are essential but unreadable
