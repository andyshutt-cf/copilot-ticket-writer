---
applyTo: "**"
---

# Figma Instructions

## Source Preference

- **Prefer local exports** in `ticket-writer-docs/design/` over live Figma URLs. Local exports are reliable, version-controlled, and available offline.
- Only use a live Figma URL when a local export does not exist or the user explicitly provides a URL and requests live retrieval.

---

## Using Figma URLs

When a Figma URL is provided:

1. If the Figma API is available via a tool (`fetch` or equivalent), use it to retrieve the file or node data.
2. If the API is not available, ask the user to export the relevant screens and place them in `ticket-writer-docs/design/` before proceeding.
3. Do not guess at screen contents or component names from a URL alone.

---

## Naming Conventions

- Always cite **exact screen names and state names** as they appear in Figma layer/frame names.
- Do **not** use colloquial or inferred names.

| ❌ Avoid | ✅ Prefer |
|---|---|
| "the login screen" | `Auth / Sign In / Default State` |
| "the error state" | `Auth / Sign In / Error — Invalid Credentials` |
| "the empty list view" | `Dashboard / Activity Feed / Empty State` |

- When describing UI behavior in Acceptance Criteria, reference the **exact Figma component name**, not a description of it.

---

## Documenting New Features in `ticket-writer-docs/design/`

For every new feature that involves UI changes, create or update a file in `ticket-writer-docs/design/` that includes:

1. **Figma screen path** — the full path to the top-level frame in Figma (e.g., `Auth / Sign Up / Default State`)
2. **Exported PNG filename** — the filename of the exported asset stored in `ticket-writer-docs/design/` (e.g., `auth-sign-up-default.png`)
3. **Component summary** — a Markdown table or list of all components on the screen and their relevant states:

```markdown
## Components

| Component | States |
|---|---|
| `PrimaryButton` | Default, Hover, Active, Disabled, Loading |
| `TextInput` | Default, Focused, Error, Filled, Disabled |
| `ErrorBanner` | Visible, Hidden |
```

4. **Accessibility annotations** — note focus order, ARIA labels, color contrast requirements, and keyboard navigation behavior.
5. **Responsive behavior** — breakpoints and layout changes (e.g., stacked vs. side-by-side, hidden elements at mobile).

---

## Flagging Discrepancies

If a Figma spec references a component, pattern, or behavior that **does not match** the existing codebase:

> ⚠️ Discrepancy detected: Figma specifies `<ComponentName>` in state `<StateName>`, but the codebase component `<CodeComponentName>` does not support this state. Clarify with design or engineering before generating acceptance criteria.

Flag the discrepancy and **do not silently reconcile** it in the ticket. Surface it for the team to resolve.
