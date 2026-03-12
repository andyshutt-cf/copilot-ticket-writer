# Building @ticket-writer: How We Built an AI Agent That Writes Better Jira Tickets

*A journey from complexity to clarity — turning messy product context into actionable engineering tickets.*

---

## The Problem We Set Out to Solve

Picture this: Your product team has gathered requirements from three separate documents, a Figma design system, and scattered transcripts from various 3-Amigos sessions. A developer is ready to start work, but the ticket sitting in Jira is vague. It references undefined terms, lacks clear acceptance criteria, and is missing critical technical context. The developer has to spend hours hunting through documents just to understand what's being asked.

Sound familiar?

We realised that writing good Jira tickets is a craft — one that requires balancing product intent, design requirements, technical constraints, and team conventions all at once. It's not something you can do well from an empty room. But what if you could?

---

## The Challenge: Context, Consistency, and Scale

As we worked with engineering and product teams across multiple projects, we noticed a pattern:

**The best tickets come from teams that:**
- Have a shared understanding of what *good* looks like (format, structure, language)
- Ground the ticket in actual codebase patterns, not wishful thinking
- Use consistent terminology and acronyms across the entire backlog
- Ask clarifying questions *after* presenting a draft (not before)
- Balance detail with brevity — no wall-of-text tickets

**But creating these tickets took time.** Lots of time. A single, high-quality ticket required:
1. Reading and synthesising multiple documents
2. Understanding the team's domain and conventions
3. Researching the codebase to reference actual modules and patterns
4. Structuring the output in the team's preferred format
5. Asking the right follow-up questions

For one ticket, this could mean 45 minutes of work. For a whole backlog, it's unsustainable.

---

## The Insight: AI Agents Thrive on Explicit Rules

We started sketching out what a "perfect ticket writer" would need:

- **A system prompt** that captured the philosophy: draft-first, ground in reality, write in UK English, follow our structure religiously
- **Rules files** that defined ticket formats, acceptance criteria styling (BDD/Gherkin, first-person only), and when each ticket type applies
- **A knowledge document** where teams could describe their company, squad, dependencies, and glossary of terms
- **Access to codebase** so the agent could reference real modules, naming conventions, and existing patterns
- **Support for context**: transcripts, design exports, PDFs — everything that informs a good ticket

The more explicit and structured the inputs, the better the output.

---

## The Solution: The @ticket-writer Agent

We built `@ticket-writer` as a custom GitHub Copilot agent. Here's what makes it work:

### 1. **Structured Knowledge Input**

The agent loads two critical documents before generating *any* ticket:

- **aacf-ticket-writer-requirements.md** — Authoritative rules for structure, formatting, and behaviour. These rules always win.
- **aacf-knowledge-document.md** — Team context: company background, squad responsibility, dependencies, constraints, and a glossary of acronyms.

Teams fill in a template with their own details. The more thorough the template, the better the tickets.

### 2. **Codebase Integration**

The agent uses `#codebase` to:
- Identify existing patterns and conventions in the affected area
- Reference actual module boundaries and API contracts
- Call out naming conventions the team uses for files, classes, functions, and variables
- Surface test strategy and existing test coverage

This grounds the ticket in *what actually exists*, not fantasy.

### 3. **Draft-First Workflow**

The agent generates a complete ticket immediately, making reasonable, clearly stated assumptions where details are missing. No waiting. No "I need more information before I can help."

After the draft, it asks clarifying questions. This flips the traditional ticket-writing flow and dramatically improves feedback loops.

That draft-first path remains the default mode. When a user wants the agent to interview first instead, they can switch modes explicitly with `Mode: interactive` in the chat request.

In interactive mode, the agent scans the codebase and docs first, asks an initial batch of 3-6 numbered clarification questions with A/B/C options, and waits before drafting. The user can include `generate` at any time to stop the interview and produce a draft from the information collected so far.

### 4. **Four Ticket Types, One Philosophy**

- **Story**: Full structure (context, user story, analytics, design, tech notes, ACs)
- **Bug**: Steps to replicate, actual/expected results, ACs
- **Task**: Tech-focused; shorter than stories
- **Spike**: Time-boxed investigations with success criteria

Each has a distinct structure, but all follow the same principles: clarity, specificity, no vague language.

### 5. **Strict Acceptance Criteria Format**

This was non-negotiable. The agent generates ACs in BDD/Gherkin style:

```
**GIVEN** I am logged in
**WHEN** I click "Save"
**THEN** I see a success message
**AND** the data is persisted to the database
```

First-person only. Gherkin keywords bold and ALL CAPS. No other bold text. This makes ACs machine-readable *and* human-readable.

---

## The Build Process

Here's how we arrived at this solution:

### Phase 1: Gathering Requirements

We conducted 3-Amigos sessions with product managers, engineers, and QA leads. We asked:
- What frustrates you about tickets you receive?
- What does a *great* ticket look like for your team?
- What terminology and conventions do we need people to understand?

We captured everything in transcripts and requirements documents.

### Phase 2: Extracting Core Rules

From those conversations, we extracted the non-negotiables:
- UK English throughout (behaviour, colour, authorise)
- No vague language ("works as expected" is banned)
- Acceptance criteria must be testable and specific
- Tech Considerations should raise concerns, not prescribe solutions
- Analytics and design must be considered from the start

We documented these as rules, not suggestions.

### Phase 3: Building the Knowledge Template

We created a template that teams fill in:
1. Company / Client Background (what does your product do?)
2. Team / Squad Description (what do you own, and what's out of scope?)
3. Relevant Context & Documentation (dependencies, constraints, PRDs, API specs)
4. Glossary of Terms & Acronyms (domain and technical terminology)

The better this is filled in, the better the tickets.

### Phase 4: Creating the Agent System Prompt

We wrote a detailed system prompt that combines all of the above: the rules, the knowledge, the behaviour model, and the output formats. The agent loads knowledge documents, grounds in codebase patterns, and follows the ticket structure exactly.

### Phase 5: Testing and Iteration

We tested the agent with real teams, real projects, and real constraints:
- Could the agent ground in an unfamiliar codebase quickly?
- Did the tickets actually help developers ship faster?
- Were clarifying questions helpful or annoying?
- Did the BDD/Gherkin format work in practice?

We iterated until the tickets consistently passed our quality bar.

---

## Key Insights from Building This

### 1. Explicitness Scales

The more explicit the rules and knowledge, the better the output. Vague instructions produce vague tickets. Crystal-clear rules produce high-quality tickets, consistently.

### 2. Context Is Everything

A ticket without team context is guesswork. Grounding in the codebase, the knowledge document, and previous decisions makes all the difference. The agent can't know your domain — but if *you* tell it, it will use it.

### 3. Draft-First Wins

Starting with a full draft (even with assumptions) is faster than asking for requirements upfront. The user can correct the draft by reference, not from scratch.

### 4. Structure Is Invisible

Good structure isn't something the reader notices — it's something they *need*. Acceptance criteria in BDD format, sections in a fixed order, consistent language — these feel constraining until you try them, then they feel obvious.

### 5. Teams Have Different Needs

No single ticket structure works for everyone. But *any* structure, applied consistently, beats ad-hoc chaos. The agent adapts by ticket type and team context, not by inventing new formats.

---

## How @ticket-writer Works in Practice

When a user invokes the agent in VS Code:

```
@ticket-writer, write me a ticket for a new campaign feature
```

The agent:

1. **Loads the knowledge document** to understand the company, squad, and team terminology
2. **Loads the requirements rules** to understand the exact structure it must follow
3. **Scans the codebase** using `#codebase` to find relevant modules, naming conventions, and test patterns
4. **Drafts a complete ticket** in Markdown, following the Story/Bug/Task/Spike template
5. **Outputs the draft immediately** with clearly stated assumptions
6. **Asks clarifying questions** to refine and validate

If the user prefers an interview-first workflow, they can invoke:

```
@ticket-writer Mode: interactive Write me a ticket for a new campaign feature
```

In that mode, the agent still loads the same knowledge and codebase context, but it asks targeted clarification questions before producing the first draft.

The user can then:
- Edit the draft in Copilot Chat
- Ask follow-up questions
- Post to Jira directly (only on explicit confirmation)

---

## The Artifact: What We Delivered

The ticket-writer repository includes:

```
.github/
  agents/
    ticket-writer.agent.md          # Agent definition
  ticket-writer-rules/
    ticket-format.instructions.md   # Ticket structure rules
    knowledge-ingestion.instructions.md  # Document loading workflow
    figma.instructions.md           # Design asset handling

ticket-writer-docs/
  knowledge/                        # Templates for company/squad context
  transcripts/                      # 3-Amigos session notes
  design/                           # Figma exports and summaries
  ai/                               # Copilot-generated summaries
```

Teams can fork this, fill in their own knowledge document, and start using `@ticket-writer` immediately. The agent has direct access to their codebase, so tickets are grounded in reality from day one.

---

## Lessons for Building AI Agents

Looking back, a few principles emerged:

1. **Make the rules explicit.** The more rules the agent has, the more consistent the output. Rules feel restrictive but they're liberating — both for the agent and the user.

2. **Ground in external context.** An agent with access to real codebase, real documents, and real team knowledge produces vastly better output than one working from thin air.

3. **Optimise for iteration, not perfection.** Generate a draft immediately and ask clarifying questions after. This mirrors how humans actually work.

We kept that as the default, but one thing we learned is that explicit mode selection matters. A plain-text directive such as `Mode: interactive` is clearer for agent behaviour than making a chat instruction look like a shell flag.

4. **Consistency beats cleverness.** A team that has consistent structure, language, and format benefits more than a team with beautifully varied tickets. Structure is invisible when it works.

5. **Let teams be themselves.** Provide templates, but make customisation easy. The best knowledge document is one that reflects your actual team, not a generic template.

---

## What's Next?

We're exploring:
- **Jira API integration**: Post tickets directly from the agent without copy-paste
- **Figma deep-linking**: Reference design tokens and components directly
- **Multi-team support**: Manage multiple knowledge documents for different squads
- **Ticket refinement**: Use the agent to split epics or refine existing tickets

---

## Try It Yourself

If you're interested in using `@ticket-writer` for your team:

1. Clone the repository
2. Copy `.github/` and `ticket-writer-docs/` to your project
3. Fill in `ticket-writer-docs/knowledge/aacf-knowledge-document.md` with your team's context
4. Customise the formatting rules if needed (in `.github/ticket-writer-rules/`)
5. Start a chat with `@ticket-writer` in VS Code

The agent will immediately have access to your codebase and your team's knowledge.

---

## Conclusion

Writing good Jira tickets is hard. It requires context, clarity, and consistency. But when you make the rules explicit, ground the agent in real team knowledge and real codebase patterns, and optimise for iteration, you can build an agent that generates tickets teammates actually want to read.

The goal isn't to replace people — it's to augment their thinking. To turn a blank page into a structured draft with assumptions called out clearly. To free people from tick-box ticket writing and let them focus on the stuff that actually requires thought.

`@ticket-writer` does that. We hope it helps your team ship faster.

---

*Built by the engineering team at AACF. Open-sourced because better tickets make faster shipping.*
