## Plan: VS Code Copilot Agent "@ticket-writer"

Build a VS Code custom agent that reliably turns project context (codebase + provided docs + Figma specs) into a Jira-ready Markdown ticket. Keep the default workflow offline/VS Code-only (no web UI dependencies) and make Jira posting optional via MCP.

**Steps**
1. Create repository scaffolding for Copilot customization
   1. Add workspace agent at `.github/agents/ticket-writer.agent.md`.
   2. Add always-on workspace guidance at `.github/copilot-instructions.md`.
   3. Add targeted instruction files under `.github/instructions/` for ticket format + knowledge ingestion + Figma usage.
   4. (Optional) Add a prompt file under `.github/prompts/` to run the workflow as a slash command (for example, `/ticket`).

2. Establish a durable "knowledge ingestion" workflow (PDFs, transcripts, Figma)
   1. Create `docs/knowledge/` and store extracted text/Markdown versions of PDFs there.
   2. Create `docs/transcripts/` and store 3-Amigo notes as Markdown with a consistent header (date, attendees, decisions, open questions).
   3. Create `docs/design/` for Figma exports (key screens to PNG plus a short Markdown summary of components/states).
   4. Create `docs/ai/` to store Copilot-produced summaries so the agent can reference concise, curated context instead of re-parsing large sources each time.

3. Define the ticket output contract (single source of truth)
   1. Standardize the Jira ticket Markdown structure: Summary, User Story, Acceptance Criteria (Gherkin), Technical Notes, Impact Analysis.
   2. Define required vs optional fields (labels, components, rollout plan, telemetry).
   3. Include a "missing info" behavior: the agent must ask 1–3 targeted questions when critical inputs are absent.

4. Implement contextual awareness rules in the agent
   1. Always begin by determining whether code changes are implied; if yes, request `#codebase` and any specific `#file` references.
   2. If the request references a feature area, use `#codebase` (search/usages) to identify existing patterns, API boundaries, naming conventions, and test strategy.
   3. If the request references requirements/design docs, load the relevant extracted text files under `docs/knowledge/`, transcripts under `docs/transcripts/`, and design summary under `docs/design/`.
   4. Ensure the ticket reflects the codebase reality (modules, endpoints, data models) and not hypothetical structure.

5. Add Jira posting capability (optional) via MCP
   1. Choose an MCP server that wraps Jira's REST APIs (create issue, add labels/components, set description).
   2. Configure it in `.vscode/mcp.json` using environment/input variables for credentials.
   3. Update the agent tools list to include the Jira MCP tool set only when the team opts in.
   4. Add a safe default: generate Markdown first; only POST if the user explicitly confirms the Jira project and issue type.

6. Verification and iteration
   1. Validate the custom agent loads: Chat → Configure Custom Agents → ensure "ticket-writer" appears.
   2. Validate instruction loading: Chat → Diagnostics shows `.github/copilot-instructions.md` + referenced `.instructions.md` files.
   3. Run three ticket-writing scenarios:
      1. Pure requirements doc (no codebase).
      2. Existing codebase feature enhancement (uses `#codebase`).
      3. UI-driven story (uses Figma URL and/or local design exports).
   4. Confirm output conforms to the contract, and the agent asks only necessary questions.

**Relevant files**
- `.github/agents/ticket-writer.agent.md` — define the persona, tools, model preference, and handoffs.
- `.github/copilot-instructions.md` — always-on rules about where knowledge lives and how to keep it updated.
- `.github/instructions/ticket-format.instructions.md` — the exact Markdown ticket template and Gherkin conventions.
- `.github/instructions/knowledge-ingestion.instructions.md` — how to use PDFs/transcripts/design exports and when to ask for more context.
- `.github/instructions/figma.instructions.md` — how to use the Figma link vs local exports, and how to cite screen/state names.
- `.vscode/mcp.json` (optional) — Jira MCP server configuration.

**Verification**
1. Confirm VS Code version supports custom agents (1.106+).
2. Open Chat → agent picker → select the custom agent.
3. Ensure the agent can:
   1. Request `#codebase` and `#file` when needed.
   2. Produce the Jira Markdown structure consistently.
   3. Incorporate extracted knowledge documents and transcripts.
4. If MCP is enabled: dry-run by generating Markdown, then POST to a test Jira project.

**Decisions**
- Default workflow outputs Markdown only; Jira posting is optional and must be explicitly confirmed per run.
- PDFs are converted to text/Markdown and checked into the repo for reliable grounding and diffable updates.

**Further Considerations**
1. Jira field mapping: decide which fields are mandatory (project key, issue type, epic link, components, labels).
2. Knowledge doc freshness: decide how and when to regenerate extracted text (manual vs script).
3. Security: define where Jira credentials live (input variables, env vars, secret manager) and whether to enable MCP sandboxing on macOS.
