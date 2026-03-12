# ticket-writer — GitHub Copilot Custom Agent

A VS Code GitHub Copilot custom agent that turns project context — codebase, requirements docs, transcripts, and Figma designs — into Jira-ready Markdown tickets.

---

## Overview

`@ticket-writer` is a custom Copilot agent that drafts actionable Jira tickets (Story, Bug, Task, Spike) following a consistent, team-agreed structure. It grounds its output in real codebase patterns, loaded knowledge documents, and design assets — not guesswork.

**Key behaviours:**
- **Default mode:** Drafts a full ticket immediately; asks clarifying questions after (not before)
- **Interactive mode:** Starts with 3-6 numbered A/B/C questions, can ask one narrower follow-up round if needed, and lets the user stop the interview at any time with `generate` — activate with `--interactive`
- Adapts structure based on ticket type (Story / Bug / Task / Spike)
- Uses `#codebase` to reference actual modules, naming conventions, and test patterns
- Outputs Markdown by default; posts to Jira only on explicit confirmation
- Writes in UK English throughout

---

## The Story Behind @ticket-writer

Curious how this agent came together? Read the full journey in [blog.md](blog.md) — from identifying the problem (vague, context-scattered tickets) through building the solution (explicit rules, knowledge grounding, codebase integration). It covers our design decisions, the phases we went through, and key insights about building AI agents.

---

## Requirements

- **VS Code** 1.106 or later
- **GitHub Copilot** subscription with access to Custom Agents (Copilot Chat)
- **Node.js** 18+ (required for the CLI script and the optional Jira MCP integration)

---

## Repository Structure

```
.github/
  agents/
    ticket-writer.agent.md          # Agent definition (model, tools, system prompt)
  ticket-writer-rules/
    ticket-format.instructions.md   # Ticket structure rules and AC format
    knowledge-ingestion.instructions.md  # Rules for loading docs and transcripts
    figma.instructions.md           # Rules for using Figma URLs and local exports
  ticket-writer-instructions.md     # Always-on workspace rules for all Copilot interactions

ticket-writer-scripts/
  create-jira-ticket.js             # CLI script to create a Jira issue from a Markdown ticket

ticket-writer-docs/
  knowledge/                        # Extracted text from PDFs and requirement docs
  transcripts/                      # 3-Amigos session notes and meeting transcripts
  design/                           # Figma exports (PNG screens + Markdown summaries)
  ai/                               # Copilot-produced summaries for efficient context reuse

.vscode/
  mcp.json                          # Optional: Jira MCP server configuration (in-chat posting)
  tasks.json                        # VS Code tasks for running the CLI script

.env.example                        # Template for Jira credentials (copy to .env)
package.json                        # npm script wrapper for the CLI tool
```

---

## Installation

### Option A — Add to an existing project (recommended)

Copy the following into your client's project repository root:

```bash
# From this repo, copy:
cp -r .github /path/to/client-project/
cp -r ticket-writer-docs /path/to/client-project/
cp -r ticket-writer-scripts /path/to/client-project/
cp package.json .env.example /path/to/client-project/
cp -r .vscode /path/to/client-project/   # optional, only if using Jira MCP or VS Code tasks
```

The agent will then have direct `#codebase` access to the client's code, which significantly improves ticket quality.

> **Note:** Your client repo will now contain:
> - `.github/agents/` — agent definition
> - `.github/ticket-writer-rules/` — ticket format and workflow rules
> - `.github/ticket-writer-instructions.md` — workspace instructions
> - `ticket-writer-docs/` — knowledge base, transcripts, design assets, and AI summaries

### Option B — Use as a standalone workspace

Open this repository in VS Code and reference client files manually using `#file` when invoking the agent. Useful for quick ticket drafting without modifying the client's repo.

---

## Configuration

### 1. Fill in the Knowledge Document

The knowledge document is the primary way to customise the agent for a specific client or squad. Open and complete:

```
docs/knowledge/aacf-knowledge-document.md
```

Fill in all sections:
- **Company / Client Background** — what the product does, who uses it
- **Team / Squad Description** — what the squad owns, what's out of scope
- **Relevant Context & Documentation** — dependencies, constraints, supporting docs
- **Glossary of Terms & Acronyms** — all domain-specific and internal terms

The more detail provided, the higher the quality of the generated tickets.

### 2. Customise the Ticket Format (if required)

If the client uses a different ticket structure than the default, edit:

```
.github/ticket-writer-rules/ticket-format.instructions.md
```

This file controls:
- The sections included in each ticket type (Story, Bug, Task, Spike)
- The Acceptance Criteria table format and Gherkin rules
- Language and quality requirements

The ticket format for each client can vary. When switching between clients, update both:
1. `ticket-writer-docs/knowledge/aacf-knowledge-document.md` — client context
2. `.github/ticket-writer-rules/ticket-format.instructions.md` — ticket structure (if different)

### 3. Add Knowledge Documents (optional but recommended)

Place extracted requirement documents, PRDs, or process docs in `ticket-writer-docs/knowledge/`. The agent loads these automatically when generating tickets.

**To extract a PDF:**
```bash
pip install pdfminer.six
python3 -c "
import sys
from pdfminer.high_level import extract_text
print(extract_text(sys.argv[1]))
" "your-document.pdf" > ticket-writer-docs/knowledge/your-document.md
```

Add a header to each extracted file:
```markdown
# Document Title
> Source: original-filename.pdf
> Extracted: YYYY-MM-DD
```

### 4. Add Transcripts (optional)

Store 3-Amigos session notes and meeting transcripts in `ticket-writer-docs/transcripts/`. Use this header format:

```markdown
# Session Title
> Date: YYYY-MM-DD
> Attendees: Name (Role), Name (Role)

## Decisions
- 

## Open Questions
- 
```

### 5. Add Design Exports (optional)

Export key Figma screens to `ticket-writer-docs/design/` as PNG files alongside a Markdown summary. Name files to match the Figma layer path, using `--` as a separator:

```
ticket-writer-docs/design/auth--sign-up--default.png
ticket-writer-docs/design/auth--sign-up--default.md
```

### 6. Enable Jira Integration

There are two ways to post tickets to Jira. Both read the same credentials — choose whichever suits your workflow.

#### Option A — CLI script (recommended)

The CLI script posts a saved Markdown file directly to Jira from your terminal:

1. Copy `.env.example` to `.env` and fill in your credentials:

```bash
cp .env.example .env
```

```ini
JIRA_BASE_URL=https://yourorg.atlassian.net
JIRA_EMAIL=your.email@yourorg.com
JIRA_API_TOKEN=your-api-token-here
JIRA_PROJECT_KEY=PROJ
```

> Generate an API token at [id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens)

2. Save the generated ticket Markdown to a file (e.g. `ticket.md`), then run:

```bash
node ticket-writer-scripts/create-jira-ticket.js --file ticket.md
```

Or use the built-in VS Code task: **Terminal → Run Task → Create Jira Ticket from File**

See the [CLI Script Reference](#cli-script-reference) section below for all available options.

#### Option B — MCP server (in-chat posting)

The Jira MCP server lets the agent post directly from within Copilot Chat. VS Code will prompt you for credentials at runtime — nothing is hardcoded.

Edit `.vscode/mcp.json` to configure the server, then confirm credentials when prompted by VS Code.

You will need:
- Your Jira instance URL (e.g. `https://yourorg.atlassian.net`)
- Your Jira email address
- A Jira API token — generate one at [id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens)

> The agent always generates Markdown first. It will only POST to Jira if you explicitly confirm (e.g. "yes, post this to Jira").

---

## Usage

### Starting the agent

1. Open **GitHub Copilot Chat** in VS Code (`Ctrl+Shift+I` / `Cmd+Shift+I`)
2. Click the agent picker (the `@` icon or dropdown)
3. Select **ticket-writer**
4. Describe your feature, bug, or task in plain language

### Example prompts

The agent has two modes. Use the one that best fits the situation.

#### Default mode — draft immediately

**Story ticket:**
```
@ticket-writer Add a "Remember me" option to the login screen so users 
don't have to re-authenticate on each visit.
```

**Bug ticket:**
```
@ticket-writer Bug: the password reset email is not being sent when the 
user's email address contains uppercase letters.
```

**Task ticket:**
```
@ticket-writer Task: update the Node.js version in the CI pipeline from 
18 to 20 across all services.
```

**Spike ticket:**
```
@ticket-writer Spike: investigate whether we can replace our current PDF 
generation library with a lighter-weight alternative that supports CSS Grid.
```

**With codebase context:**
```
@ticket-writer #codebase Add rate limiting to the /api/auth/login endpoint. 
Use the same pattern as other protected routes.
```

**With a specific file:**
```
@ticket-writer #file:src/components/LoginForm.tsx Update the login form 
to show inline validation errors rather than a toast notification.
```

**With a Figma design:**
```
@ticket-writer Implement the updated dashboard header. 
Figma: https://www.figma.com/file/...
```

#### Interactive mode — interview first, then draft

Add `--interactive` to the prompt to switch the agent into an interview-first flow. The agent will scan the codebase and docs, ask an initial batch of 3-6 numbered clarification questions (each with **A**, **B**, and **C Custom** options), and wait for your answers before drafting.

If material gaps remain, it may ask one additional round of up to 3 narrower questions. You can stop the interview at any time by including `generate` in your reply, and the agent will draft using the information collected so far.

```
@ticket-writer --interactive Add a "Remember me" option to the login screen.
```

```
@ticket-writer --interactive #codebase Add rate limiting to the /api/auth/login endpoint.
```

> **Important:** `--interactive` is a prompt-level mode token, not a real parsed shell or operating-system CLI flag. It is detected as a literal string in the Copilot chat message. If a future CLI wrapper is added, it can convert a real `--interactive` argument into this token before invoking the agent.

### Workflow

#### Default mode

1. The agent drafts a full ticket immediately
2. Review the draft — it will clearly state any assumptions made
3. The agent follows up with up to 3 clarifying questions
4. Answer the questions to refine the ticket
5. Post the ticket to Jira using one of the methods below:
  - **CLI script:** save the Markdown to a file, then run `node ticket-writer-scripts/create-jira-ticket.js --file ticket.md`
   - **VS Code task:** `Terminal → Run Task → Create Jira Ticket from File`
   - **MCP (in-chat):** confirm to the agent with "yes, post this to Jira" (requires MCP setup)

---

## CLI Script Reference

`ticket-writer-scripts/create-jira-ticket.js` creates a Jira issue from a Markdown ticket file generated by the agent.

### Usage

```bash
# Create an issue from a file
node ticket-writer-scripts/create-jira-ticket.js --file ticket.md

# Pipe Markdown from stdin
cat ticket.md | node ticket-writer-scripts/create-jira-ticket.js

# Override the project key
node ticket-writer-scripts/create-jira-ticket.js --file ticket.md --project ACME

# Override the issue type (auto-detected if omitted)
node ticket-writer-scripts/create-jira-ticket.js --file ticket.md --type Bug

# Preview the request without creating the issue
node ticket-writer-scripts/create-jira-ticket.js --file ticket.md --dry-run

# npm script shorthand
npm run create-ticket -- --file ticket.md
```

### Options

| Flag | Description |
|------|-------------|
| `--file <path>` | Path to a Markdown ticket file. Defaults to stdin if omitted. |
| `--project <key>` | Jira project key (e.g. `ACME`). Overrides `JIRA_PROJECT_KEY`. |
| `--type <type>` | Issue type: `Story`, `Bug`, `Task`, or `Spike`. Auto-detected from the Markdown if omitted. |
| `--dry-run` | Validates and previews the payload without creating the issue. |
| `--help` | Prints usage information. |

### Issue type detection

If `--type` is not specified, the script infers the issue type from the Markdown structure:

| Signal | Detected type |
|--------|--------------|
| Summary starts with `Bug:` or contains `[Bug]` | Bug |
| Summary starts with `Task:` or contains `[Task]` | Task |
| Summary starts with `Spike:` or contains `[Spike]` | Spike |
| Contains `## Steps to Replicate` or `## Actual Results` | Bug |
| Contains `## Goals` and `## Success Criteria` | Spike |
| Contains `## Tech Considerations` but no `## User Story` | Task |
| Everything else | Story |

### Setup

1. Copy `.env.example` to `.env` and fill in your Jira credentials.
2. Add `.env` to your `.gitignore` (already included in this repo).
3. Run `node ticket-writer-scripts/create-jira-ticket.js --help` to verify the installation.

#### Interactive mode

1. The agent scans the codebase and loads relevant docs
2. The agent asks an initial batch of 3-6 numbered questions (A / B / C Custom options) in a single response
3. Answer any or all of the questions, and include `generate` whenever you want the ticket drafted
4. If needed, the agent may ask one narrower follow-up round of up to 3 questions
5. The agent drafts the full ticket based on your answers, stating assumptions for anything unanswered
6. Copy the Markdown into Jira, or (if MCP is enabled) confirm to post directly

---

## Ticket Structure Reference

### Story
| Section | Required |
|---|---|
| Summary / Title | ✅ |
| Background / Context | ✅ |
| User Story | ✅ |
| Analytics / Tracking | ✅ |
| Design Link | ✅ |
| Tech Considerations / Engineering Notes | ✅ |
| Acceptance Criteria | ✅ |
| Open Questions / Clarifications | If applicable |

### Bug
| Section | Required |
|---|---|
| Summary / Title | ✅ |
| Background / Context | ✅ |
| Steps to Replicate | ✅ |
| Actual Results | ✅ |
| Desired Outcome | ✅ |
| Acceptance Criteria | ✅ |
| Open Questions / Clarifications | If applicable |

### Task
| Section | Required |
|---|---|
| Summary / Title | ✅ |
| Background / Context | ✅ |
| Design Link | If applicable |
| Tech Considerations / Engineering Notes | ✅ |
| Acceptance Criteria | ✅ |
| Open Questions / Clarifications | If applicable |

### Spike
| Section | Required |
|---|---|
| Summary / Title | ✅ |
| Background / Context | ✅ |
| Goals | ✅ |
| Success Criteria | ✅ |
| Expected Outcome / Deliverable | ✅ |
| Full Product List | ✅ |
| Open Questions / Clarifications | If applicable |

### Acceptance Criteria format

All ACs use a Markdown table with Dev and QA sign-off columns:

```markdown
| AC # | Scenario | Dev ✅ | QA ✅ |
|------|----------|--------|-------|
| 1 | **GIVEN** I am logged in<br>**WHEN** I click "Save"<br>**THEN** I see a success message | [ ] | [ ] |
```

Rules:
- Written in **first person** — always "I …", never "user", "they", or "the system"
- Only `GIVEN`, `WHEN`, `THEN`, `AND` are ALL CAPS and bold
- No other words in the scenario should be bold or uppercased

---

## Verification

After installation, verify the agent is working correctly:

1. Open Copilot Chat → agent picker → confirm **ticket-writer** appears in the list
2. Run a default-mode test: `@ticket-writer Story: add a loading spinner to the submit button`
3. Confirm the output:
   - Follows the correct Story structure
   - Uses UK English
   - AC section is a Markdown table (not a code block)
   - ACs are written in first person
4. Run an interactive-mode test: `@ticket-writer --interactive Story: add a loading spinner to the submit button`
5. Confirm the agent:
   - Does **not** draft a ticket immediately
  - Asks an initial batch of 3-6 numbered questions with A, B, and C Custom options
  - Tells you that you can include `generate` to stop the interview and draft immediately
  - Waits for your answers before producing the ticket, unless you choose to end the interview

If the agent doesn't appear, check that your VS Code version is 1.106 or later and that you have a Copilot subscription with Custom Agents enabled.

---

## Switching Between Clients

When moving to a new client engagement, update these two files:

1. **`ticket-writer-docs/knowledge/aacf-knowledge-document.md`** — replace with the new client's context
2. **`.github/ticket-writer-rules/ticket-format.instructions.md`** — update if the client uses a different ticket structure

Keep a copy of each client's versions outside the repo (e.g. in a notes folder or separate branch) so you can restore them quickly when switching back.
