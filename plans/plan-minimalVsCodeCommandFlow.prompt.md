## Plan: Minimal VS Code Command Flow

Add the smallest native VS Code command flow that helps users launch the ticket-writer agent without remembering prompt syntax. Because this repository does not currently contain an extension scaffold or command registrations, the minimal viable command-palette solution is a tiny VS Code extension that collects a request, builds the correct `@ticket-writer` prompt, copies it to the clipboard, and opens or directs the user to Copilot Chat for submission. This keeps implementation scope low while avoiding risky assumptions about unsupported chat-injection APIs.

**Steps**
1. Confirm the command-flow scope and keep it intentionally narrow: support only two user actions in the command palette, `Ticket Writer: Draft Ticket` and `Ticket Writer: Interview First`, with no Jira-posting logic and no changes to the existing Markdown-to-Jira CLI. This preserves separation between ticket generation and Jira publishing.
2. Add a minimal VS Code extension scaffold to the repository because native command-palette entries cannot be created from the current repo shape alone. This includes extension metadata in `/Users/andy.shutt/Documents/Projects/copilot-ticket-writer/package.json` and a single extension entry file such as `/Users/andy.shutt/Documents/Projects/copilot-ticket-writer/extension.js` or `/Users/andy.shutt/Documents/Projects/copilot-ticket-writer/src/extension.ts`. This step blocks all later command work.
3. Register two commands in the extension manifest and implementation: one for default draft-first mode and one for interview-first mode. Keep them explicit rather than adding an initial mode picker so the first version is fast to discover and requires fewer interaction steps.
4. Implement the command handlers to ask the user for the ticket description with `showInputBox`, then construct the final prompt string. Recommended outputs:
   - Default command builds `@ticket-writer <description>`
   - Interactive command builds `@ticket-writer Mode: interactive <description>`
   Keep prompt construction centralized in one helper so future changes to directive syntax affect one place only.
5. Decide the command handoff behavior. Recommended MVP: copy the constructed prompt to the clipboard and show an information message telling the user to paste it into Copilot Chat. Treat automatic chat submission as out of scope for the first iteration unless a supported Copilot Chat API is verified. This avoids building on undocumented integration points.
6. Optionally improve ergonomics without expanding scope by attempting to focus Copilot Chat after copying the prompt, but only if a supported VS Code command or API is available. If that capability is not verifiably supported, fall back to the clipboard-only flow and document the manual paste step explicitly.
7. Update `/Users/andy.shutt/Documents/Projects/copilot-ticket-writer/README.md`, `/Users/andy.shutt/Documents/Projects/copilot-ticket-writer/.github/ticket-writer-instructions.md`, and `/Users/andy.shutt/Documents/Projects/copilot-ticket-writer/blog.md` to document the new launch path. Explain that users can either type prompts directly in chat or use the command palette for a guided launcher.
8. Keep `/Users/andy.shutt/Documents/Projects/copilot-ticket-writer/.github/agents/ticket-writer.agent.md` unchanged unless the prompt directive itself needs further clarification. The agent already understands `Mode: interactive`, so the wrapper should adapt to the existing behavior rather than redefining it.
9. Verify the extension packaging and the user flow end to end: command appears in the palette, input box accepts a description, the expected prompt lands on the clipboard, and the resulting prompt works in Copilot Chat for both modes. This depends on steps 2-7.

**Relevant files**
- `/Users/andy.shutt/Documents/Projects/copilot-ticket-writer/package.json` — must become an extension manifest or be expanded to include VS Code extension metadata, activation events, and command contributions.
- `/Users/andy.shutt/Documents/Projects/copilot-ticket-writer/extension.js` — recommended minimal extension entry point for registering command handlers and building prompts.
- `/Users/andy.shutt/Documents/Projects/copilot-ticket-writer/README.md` — user-facing command-flow documentation and examples.
- `/Users/andy.shutt/Documents/Projects/copilot-ticket-writer/.github/ticket-writer-instructions.md` — workspace guidance for invoking the agent manually or via the new command launcher.
- `/Users/andy.shutt/Documents/Projects/copilot-ticket-writer/blog.md` — optional product narrative update so the documented UX matches the actual repo.
- `/Users/andy.shutt/Documents/Projects/copilot-ticket-writer/.vscode/tasks.json` — intentionally out of scope for the first command-palette implementation unless you also want a task-based fallback.
- `/Users/andy.shutt/Documents/Projects/copilot-ticket-writer/ticket-writer-scripts/create-jira-ticket.js` — explicitly out of scope because Jira posting is a downstream action.

**Verification**
1. Confirm the repo contains a valid minimal VS Code extension manifest and entry point.
2. Confirm `Ticket Writer: Draft Ticket` appears in the command palette and copies a prompt of the form `@ticket-writer <description>`.
3. Confirm `Ticket Writer: Interview First` appears in the command palette and copies a prompt of the form `@ticket-writer Mode: interactive <description>`.
4. Confirm pasting each generated prompt into Copilot Chat triggers the expected draft-first or interview-first behavior with the existing agent.
5. Confirm the README and workspace instructions explain the command-based launch path and still document the direct chat prompt form as a fallback.
6. Confirm no documentation implies that the Jira CLI or VS Code tasks are responsible for selecting agent mode.

**Decisions**
- Included scope: native command-palette launcher for constructing prompts for the existing Copilot custom agent.
- Excluded scope: direct Jira posting, chat auto-submission via undocumented APIs, webview UI, multi-step form UX, and changes to the ticket-writing behavior contract.
- Recommended MVP UX: two explicit commands instead of one command plus a mode picker, because it reduces implementation complexity and user clicks.
- Recommended handoff: clipboard copy plus a clear prompt to paste into Copilot Chat, because it is the safest integration point with the current repo and tool surface.

**Further Considerations**
1. If a supported Copilot Chat API exists for pre-filling or sending prompts, add that in a later iteration and keep the clipboard fallback as a safe backup.
2. If users want richer structure later, evolve the launcher into a small webview form with fields for mode, description, context scope, and optional ticket type.
3. If extension packaging is too heavy for this repo, the fallback is a task-driven prompt builder script, but that is less native than the command-palette flow the user asked for.