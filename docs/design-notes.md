# Teaching Assistant — Design Notes (brainstorm)

Status: brainstorming. Nothing built yet.
Source of truth for decisions so far and for **open questions to raise at the start of the next session**.

---

## Decided

### Product
- Parents upload photos / screenshots / PDFs of what was taught; AI generates printable worksheets
  following the Master Instructions (Grade 1 ICSE rules: class-material-first, progression, no answers
  on student pages, A4, generous writing space, etc.).
- Fully autonomous: no human review step. Human input only as a last resort (a tiny targeted question).
- Intended for other parents too, not only personal use.
- Handwriting (often the child's) is handled by design, not review: the system needs concepts, formats
  and school wording — not the child's exact words. Teacher/printed text is trusted; child writing is weak
  evidence; teacher marks (ticks, corrections) are strong signal; confidence scoring; cross-upload
  corroboration; photo-quality check at upload.

### Knowledge layers
- **Knowledge base** (scraped/curated, shared, offline-refreshed): keyed by country → board → grade →
  subject. Shapes generation (scope guard, formats, terminology, handwriting tie-breaks) but never adds
  concepts.
- **Session context**: what the user uploads now — primary source.
- **Child memory**: accumulated summaries across sessions (comes back via storage, see below).
- Precedence: session > child memory > knowledge base.

### Terminology bridge
- When the school switches terms (e.g. "naming words" → "nouns"), add a recap page linking old and new,
  use both terms for a few packs, then fade to the new one. Only bridge terms the school has introduced.

### Start small (v1)
- Upload lesson material → generate worksheets. User says how many.
- 1 worksheet = 1 A4 page; answer key separate at the end and not counted.

### Architecture
- **Skill** (pedagogy: master rules, workflow, page planning, exercise-type guide) + **MCP server**
  (storage, PDF rendering, layout checks).
- Claude in the chat does all thinking/generation; the MCP has no AI and runs on the parent's Claude plan.
- "Exercise types" = fixed layout templates (circle/tick, match, fill blanks, rewrite, write answer,
  count-and-write…). Content comes only from uploaded material; set grows from real material. No scraping.
- Claude emits structured worksheet JSON → MCP renders A4 PDF and runs layout checks.

### Storage (Hugging Face)
- Uploads and generated sheets saved to a private HF bucket, organised per user, so material need not be
  re-uploaded for revision.
- Separate **lessons** (uploads + summary), **clean copies**, and **packs** (each generated PDF):
  ```
  users/<user_id>/<child>/
    lessons/<lesson_id>/ uploads/ · summary.json · clean/
    packs/<pack_id>/     pack.json · pack.pdf
    index.json
  ```
- Multiple PDFs per session (e.g. practice now, revision later — same or different session).

### Model
- Recommend Opus 5.5 + High effort. Skill cannot set it; warn the user when Claude can tell it's not
  Opus 5.5. Effort can't be reliably detected → mention in setup guide.

### Classwork Practice (clean copies of school worksheets)
- Before generating **any** PDF, check stored lessons for school worksheets relevant to the topics and
  ask whether to include clean copies (answers removed). One clean copy can be reused across many PDFs.
- Clean copies are **extra** — not counted toward the requested number of worksheets.
- Labelled in the PDF as **"Classwork Practice"** (e.g. "Classwork Practice — Worksheet 22").
- Answer key: Claude solves, then compares with teacher marks on the original.
  **On disagreement → flag it and let the parent choose.**

### Images
- Always use standard free clip art / icons that a child instantly recognises (curated, openly licensed
  icon set bundled with the MCP). If no suitable image exists, reword the question so it needs no picture.
  (Licences of the chosen set to be verified at build time.)

---

## Open questions — ask the user next session (do not assume)

**Scope & users**
1. Which subjects? (Hindi / other Indian languages affect fonts and writing lines.)
2. What handwriting lines does the school use — 4-line, 2-line, plain ruled?
3. Typical batch size — one chapter (5–15 photos) or a whole term at once?
4. Launch scope — India only (CBSE/ICSE) first, or multi-country?
5. Just you/family first (simple token setup) or other parents from day one (needs login/OAuth)?
6. Per-child folders (parents with 2+ kids) or per user only?
7. Pooling material across parents of the same class — interesting or off the table?

**Inputs & behaviour**
8. Ask parents for school name + textbook series, or infer from uploads?
9. Thin material (e.g. one photo) — refuse, or fill gaps from the knowledge base and mark as general practice?
10. If the number of worksheets isn't given — ask, or default (e.g. 5)?
11. Wrong model detected — warn and continue, or ask "switch or continue?"
12. Placement of Classwork Practice pages — mixed into the progression by concept level (A), or grouped
    in one section just before the assessment (B, recommended)? *(User asked for clarification; not decided.)*
13. How the parent is shown teacher-vs-Claude answer disagreements (e.g. a short list in chat before the
    answer key is finalised) and whether their choice is remembered.

**Output & cost**
14. Printed PDF only, or also an on-screen/tablet version later?
15. Is pay-per-use API cost acceptable (relevant if the server ever calls Claude itself)?

**Next steps**
16. Collect 2–3 real photo sets (English, Maths, EVS) as first test cases?
17. Ready to start building? Suggested first step: prove the photo hand-off from chat to the MCP server.
