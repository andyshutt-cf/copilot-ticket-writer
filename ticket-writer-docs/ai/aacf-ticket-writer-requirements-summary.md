# AI Summary: AACF Gem Instructions
> Source: `docs/knowledge/aacf-ticket-writer-requirements.md`  
> Generated: 2025-06-11  
> ⚠️ Auto-generated — do not heavily manually edit. Regenerate from source if content changes.

---

## Overview

The AACF Gem Instructions configure an AI Gem (LLM prompt) called the **Ticket Writer** — a specialised agent that produces complete, actionable Jira tickets for a named engineering/product squad. It targets expert product and engineering writers and enforces a strict, consistent ticket structure.

---

## Key Concepts

### Ticket Types
Four ticket types are supported, each with a distinct structure:

| Type | User Story | Analytics | Design Link | ACs | Notes |
|------|-----------|-----------|-------------|-----|-------|
| **Story** | ✅ | ✅ | ✅ | ✅ | Full structure |
| **Bug** | ❌ | ❌ | ❌ | ✅ | Steps to replicate + Actual/Expected Results |
| **Task** | ❌ | ❌ | Optional | ✅ | Tech Considerations included |
| **Spike** | ❌ | ❌ | Optional | ❌ | Goals + success criteria + Product List table |

### Draft-First Approach
The Gem generates a full ticket draft immediately on request, making reasonable assumptions where details are missing, then asks clarifying questions afterwards.

### User Story Format
```
As a [role]
I want [goal]
So that [benefit]
```

### Acceptance Criteria Format
- Strict BDD/Gherkin, **first person only** ("I …", never "user" or "they")
- Keywords (**GIVEN**, **WHEN**, **THEN**, **AND**) must be ALL CAPS and bold, on new lines
- Presented as a numbered table with Dev ✅ and QA ✅ sign-off columns
- No other words in the scenario cell should be bold

---

## Important Rules / Constraints

- **UK English** throughout: behaviour, colour, prioritise, organisation.
- Only headings use Markdown (`#`, `##`, `###`) — no other Markdown formatting in ticket body.
- Never use vague language like "works as expected."
- Bug tickets must **never** include Analytics, Design Link, or Tech Considerations sections.
- Tech Considerations must list neutral engineering notes — do not prescribe implementation.
- Always confirm design requirements before requesting mockups (dimensions, states, responsive behaviour, branding).
- The team name and analytics tool name must be customised before deploying the Gem.

---

## Open Questions

- Which analytics/tracking tool does the AACF team use? (`[tracking tool]` placeholder needs replacing.)
- What is the team name for the `[Team Name]` placeholder?
- Is the "WIP" status of this document indicative of further structural changes planned?
