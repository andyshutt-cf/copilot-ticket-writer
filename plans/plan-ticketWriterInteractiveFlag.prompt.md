## Plan: Ticket Writer Interactive Flag

Preserve the current immediate-draft behaviour as the default, and add an explicit `--interactive` mode switch that tells Ticket Writer to enter an interview-first flow before drafting. Because this repository currently contains agent instructions and documentation rather than a real CLI, the implementation should treat `--interactive` as a documented invocation token in the agent request, for example `@ticket-writer --interactive ...`, unless a future wrapper is added to parse shell arguments.

**Steps**
1. Update `.github/agents/ticket-writer.agent.md` so the agent supports two explicit modes. Default mode remains the current immediate-draft flow. If the incoming request includes `--interactive`, the agent must switch into `Interactive Plan Mode`: scan codebase/docs first, ask an initial batch of 3-6 numbered clarification questions, give each question `A`, `B`, and `C Custom` options, explicitly tell the user they can reply with `generate` to stop the interview, and pause for the user response.
2. Revise `.github/ticket-writer-rules/ticket-format.instructions.md` to describe the dual-mode workflow clearly. Keep `Draft First, Refine After` as the default path, and add a parallel `Interactive Mode` path triggered by `--interactive`. Define the tightened fallback rule inside interactive mode: if material ambiguity remains and the user has not said `generate`, allow one narrower follow-up question round only; after that, draft with explicit assumptions rather than remaining blocked indefinitely. This step depends on step 1.
3. Align `ticket-writer-docs/knowledge/aacf-ticket-writer-requirements.md` with the dual-mode behaviour. The requirements document should state that immediate drafting is the default, but `--interactive` switches the agent into a pre-ticket interview flow with numbered A/B/C questions, a `generate` escape hatch, and at most one follow-up question round. This step depends on steps 1-2.
4. Update `.github/ticket-writer-instructions.md` so workspace guidance explains the mode switch, including how to invoke interactive mode and how lightweight post-ticket code suggestions should be presented after ticket generation. This can run in parallel with step 3 once the mode semantics are settled.
5. Refresh `README.md` to document both invocation styles. Add examples for default mode, such as `@ticket-writer Add X`, and interactive mode, such as `@ticket-writer --interactive Add X`. Document that in the current repo this is a prompt-level flag token, not a parsed operating-system CLI flag. This depends on step 2 and can run in parallel with step 4.
6. Run a consistency review across all docs and instructions to remove any wording that implies there is only one workflow. The final state should make it obvious when the agent drafts immediately, when it asks questions first, and how `--interactive` changes behaviour. This depends on steps 1-5.

**Relevant files**
- `.github/agents/ticket-writer.agent.md` — primary behaviour contract where the mode switch logic should live.
- `.github/ticket-writer-rules/ticket-format.instructions.md` — process rules that need to describe both default and interactive flows.
- `ticket-writer-docs/knowledge/aacf-ticket-writer-requirements.md` — authoritative requirements source that must be kept in sync.
- `.github/ticket-writer-instructions.md` — workspace-level instructions for how the agent is invoked and used.
- `README.md` — user-facing documentation and examples for both modes.

**Verification**
1. Confirm the agent instructions define two modes explicitly: default immediate drafting and `--interactive` interview-first drafting.
2. Confirm the interactive path starts with 3-6 numbered questions with `A`, `B`, and `C Custom` options before ticket generation.
3. Confirm the default path still allows immediate drafting when `--interactive` is not present.
4. Confirm the README and workspace docs explain that `--interactive` is currently an invocation token in the chat request, not a real parsed shell flag, unless later backed by a CLI wrapper.
5. Confirm the fallback rule for partial answers and the `generate` escape hatch are consistent everywhere.

**Decisions**
- Preserve immediate drafting as the default behaviour.
- Add interactive behaviour only when the request includes `--interactive` in the `@ticket-writer` prompt.
- Treat `--interactive` as a prompt-level mode token, not a shell-parsed CLI flag.
- Use `--interactive` as the primary option syntax because it is short, familiar, easy to scan in examples, and there is no stronger structured option mechanism available in this repository's current custom-agent setup.
- Let the user end the interactive interview at any time by including `generate` in their reply.
- Allow at most one follow-up interactive question round when material ambiguities remain.
- Keep the post-ticket `@workspace` guidance lightweight.
- Scope remains repository instructions and docs only; this repo does not currently contain application or CLI runtime code to parse real command-line flags.

**Further Considerations**
1. If you later want a true executable flag, add a thin CLI or VS Code command wrapper that converts a real `--interactive` argument into the appropriate agent prompt before invoking Copilot.
2. If native VS Code interactive prompts cannot be reliably driven from custom-agent instructions, preserve the same behaviour using numbered A/B/C question text so the user experience stays deterministic.
