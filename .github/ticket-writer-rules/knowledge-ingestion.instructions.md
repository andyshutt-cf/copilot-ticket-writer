---
applyTo: "ticket-writer-docs/**"
---

# Knowledge Ingestion Instructions

## Source of Truth

- PDFs are converted to plain text or Markdown and stored in `docs/knowledge/` as the canonical source of truth.
- Do not rely on memory or training data when a knowledge document exists — always load the file first.
- When referencing any requirement, decision, or specification, cite the **source document filename and section** (e.g., `AACF-Knowledge Document.md § 3.2 Authentication Flow`).

---

## Directory Conventions

### `docs/knowledge/`
Converted and normalised source documents. One file per original source. Filename should match the original source document (slugified, lowercase, hyphens).

### `docs/transcripts/`
Meeting and interview transcripts. Each file must begin with the following header format:

```markdown
# <Meeting Title>

**Date:** <ISO 8601 date, e.g. 2025-03-26>
**Attendees:** <Comma-separated list of names or roles>

## Decisions
- <Bulleted list of decisions made>

## Open Questions
- <Bulleted list of unresolved questions>
```

### `docs/ai/`
Auto-generated summaries of knowledge documents. One summary per source document in `docs/knowledge/`. When a knowledge document is updated, regenerate its corresponding summary in `docs/ai/` before generating any tickets that depend on it.

### `docs/design/`
Design documentation linked to Figma. See `.github/ticket-writer-rules/figma.instructions.md` for required structure.

---

## Ticket Generation Rules

1. **Before generating a ticket**, check whether relevant documents exist in `docs/knowledge/`, `docs/transcripts/`, or `docs/design/`.
2. If relevant docs exist, load and read them in full before generating. Do not skip this step.
3. If docs are **missing** (expected file not found), flag this to the user:
   > ⚠️ No knowledge document found for `<topic>`. The ticket may be incomplete. Please add the relevant doc to `docs/knowledge/` or provide the information directly.
4. If docs are **outdated** (last commit to the file is more than 30 days ago), flag this to the user:
   > ⚠️ `<filename>` was last updated more than 30 days ago. Verify it is still accurate before proceeding.

---

## Document Update Rules

- When a knowledge document in `docs/knowledge/` is updated, also update or regenerate its summary in `docs/ai/`.
- Summaries should include: document title, last-updated date, a 3–5 sentence overview, and a bulleted list of key facts or decisions.
- Do not silently carry forward stale summaries — regenerate from the updated source.

---

## Citation Format

When referencing a knowledge document inline in a ticket or response, use this format:

```
[Source: <filename> § <Section Name or Number>]
```

Example:
```
[Source: aacf-knowledge-document.md § 4.1 User Roles]
```
