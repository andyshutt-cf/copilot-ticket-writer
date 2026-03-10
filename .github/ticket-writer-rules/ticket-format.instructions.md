---
applyTo: "**"
---

# Ticket Format Instructions

These rules govern every Jira ticket produced by the ticket-writer agent. Follow them precisely.

---

## Language & Quality

- Use professional, concise **UK English** (behaviour, colour, prioritise, organisation, etc.).
- Avoid vague terms like "works as expected."
- Only headings may use Markdown (`#`, `##`, `###`).
- Do not use bold anywhere except Gherkin keywords inside AC tables and section headings.

---

## Process: Draft First, Refine After

1. On receiving a request, **generate a full draft ticket immediately** using available information.
2. If required details are missing, make reasonable, clearly stated assumptions — do not block on questions.
3. After producing the draft, ask up to 3 targeted clarifying questions to improve accuracy.
4. Update the ticket iteratively until it is actionable by engineering and QA.

Always identify these two things before expanding:
- **Ticket Type**: Story, Task, Bug, or Spike
- **Summary / Title**: one short line describing the feature, task, or bug

---

## Story Ticket Structure

All Story tickets **must** include these sections in this order:

### 1. Summary / Title
One concise line describing the feature or change.

### 2. Background / Context
Why this work is needed, the origin of the request, relevant history, constraints, and links to docs/files.

### 3. User Story
Written as plain text (not bold):
```
As a [role]
I want [goal]
So that [benefit]
```

### 4. Analytics / Tracking
Ensure analytics are reviewed and updated to capture new or modified user interactions. Confirm with the Product team if new events, properties, or funnels are required.

### 5. Design Link
`[Design link placeholder]`

### 6. Tech Considerations / Engineering Notes
List neutral considerations: dependencies, performance, scalability, testing scope, configuration vs code-change. Do **not** prescribe implementation details.

### 7. Acceptance Criteria
See AC rules below.

### 8. Open Questions / Clarifications
List explicit uncertainties to confirm before implementation.

---

## Bug Ticket Structure

Bug tickets **must not** include Analytics / Tracking, Design Link, or Tech Considerations.

1. Summary / Title
2. Background / Context
3. Steps to Replicate – step-by-step bullet list (not Gherkin)
4. Actual Results – for each step: Step / Observed (current behaviour) / Expected (correct behaviour)
5. Desired Outcome – heading only, no descriptive sentence
6. Acceptance Criteria – BDD/Gherkin format
7. Open Questions / Clarifications

---

## Task Ticket Structure

- No User Story required.
- No Analytics / Tracking.
- Include Design Link if relevant.
- Include Tech Considerations / Engineering Notes.
- Include ACs in BDD/Gherkin format.

---

## Spike Ticket Structure

- No User Story or ACs required.
- No Analytics / Tracking.
- Provide clear goals, success criteria, and expected outcome.
- Include Design Link if relevant.
- Include Tech Considerations / Engineering Notes.
- Include Full Product List table for product impact.

---

## Acceptance Criteria Rules

- One AC table per ticket.
- Number ACs sequentially (1, 2, 3…).
- Write every step in **first person** — always "I …", never "user", "they", or "the system".
- Only the Gherkin keywords (**GIVEN**, **WHEN**, **THEN**, **AND**) must be:
  - Written in ALL CAPS
  - Bold
  - On a new line inside the Scenario cell
- **Do not bold any other words** inside the Scenario cell.

### AC Table Format

| AC # | Scenario | Dev ✅ | QA ✅ |
|------|----------|--------|--------|
| 1 | **GIVEN** I am [precondition]<br>**WHEN** I [action]<br>**THEN** I [outcome] | [ ] | [ ] |
| 2 | **GIVEN** I have [state]<br>**WHEN** I [action]<br>**AND** I [additional action]<br>**THEN** I [outcome] | [ ] | [ ] |

---

## Design / Mockup Requests

Before requesting a design, always confirm:
- Dimensions, placement, colour scheme, branding
- Visual states (default, hover, active, error, empty)
- Responsive behaviour
- Accessibility requirements

Never assume defaults.
