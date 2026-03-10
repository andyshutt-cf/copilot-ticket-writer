# Copilot Workspace Instructions

## Custom Agent: ticket-writer
This workspace includes a custom `ticket-writer` agent (`.github/agents/ticket-writer.agent.md`) for generating Jira-ready Markdown tickets. Use `@ticket-writer` to invoke it.

The agent uses codebase context, project documentation, and design assets to produce tickets grounded in actual project structure — not hypothetical code.

---

## Documentation Layout

| Directory | Contents |
|---|---|
| `ticket-writer-docs/knowledge/` | Extracted knowledge from PDFs and source documents (requirements, process guides, domain context) |
| `ticket-writer-docs/transcripts/` | Meeting notes and 3-Amigos session records. Each file should be structured with: date, attendees, decisions, and open questions. |
| `ticket-writer-docs/design/` | Figma key screens exported as PNG files, each accompanied by a Markdown summary of the screen's purpose and key interactions. |
| `ticket-writer-docs/ai/` | Copilot-produced summaries of the above docs, optimised for efficient context reuse in future prompts. |

---

## Knowledge Update Workflow
When asked to extract or update knowledge from any source document, always produce **both**:
1. The raw extracted text saved to the appropriate `ticket-writer-docs/knowledge/` or `ticket-writer-docs/transcripts/` file.
2. A concise AI summary saved to `ticket-writer-docs/ai/` with the same base filename, suffixed with `.summary.md`.

This ensures future interactions can load the lightweight summary by default and fall back to raw detail when needed.

---

## Credentials and Secrets
- **Never hardcode Jira credentials**, API tokens, or any secrets in any file.
- Always use input variables (e.g. VS Code prompt inputs) or environment variables for sensitive values.
- If a workflow or script requires Jira access, document the required environment variables in a `README` or `.env.example` file.
