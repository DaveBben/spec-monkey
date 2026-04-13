# Research Protocol (Phase 0 Detail)

Detailed step-by-step instructions for Phase 0: Deep Research. The main
SKILL.md contains the phase summary; this file has the full procedure.

---

## Step 0: Clarify Intent and Gather Context

Before researching anything, confirm you understand what the user wants and
why. Wrong assumptions about intent lead to research that answers the wrong
questions.

### Part A — Confirm What and Why

Evaluate `$ARGUMENTS` against two questions:

1. **What changes does the user want?** (the concrete change)
2. **Why is this change needed?** (the motivation, problem, or goal)

If both are clearly answerable from `$ARGUMENTS`, state your understanding
and ask the user to confirm or correct:

> "Before I research the codebase, let me confirm I understand:
> - **What**: [concrete change]
> - **Why**: [motivation or problem]
>
> Is that right, or would you adjust anything?"

If either is unclear or missing, ask specifically for the missing piece using
`AskUserQuestion`. Do not guess at the "why" — a wrong assumption about
motivation leads to a structurally correct but strategically wrong plan.

This must not feel like a bureaucratic gate. If the user's input already
answers both questions clearly, confirm and move on.

### Part B — External Context Check

After confirming what and why, ask:

> "Is there any context outside this codebase that should inform my research?
> For example:
> - Architecture documents, user stories, or specs hosted elsewhere
> - Other repositories or services that will be affected
> - Business context or constraints (e.g., 'this is a throwaway prototype',
>   'this must ship by Friday', 'we're migrating off this soon')
> - URLs or documents I should review
>
> If not, just say 'no' and I'll proceed with codebase research."

Handle responses:
- **"No" or equivalent**: Note "No external context provided" and proceed.
- **URLs provided**: Use `WebFetch` to retrieve content. Summarize findings.
- **Text context provided**: Record verbatim.
- **File paths in other repos**: If accessible, read them directly. Otherwise
  ask the user to paste relevant content. Record the repository path — it
  will be used to tag tasks with their target repository in Phase 4.
- **WebFetch fails**: Note the failure, ask the user to paste content directly.

If the feature spans multiple repositories, record all repository paths.
During Phase 4 task breakdown, set each task's `repository` field to the
repo it targets. Set `repositories` in plan.json to the list of all repos.
For single-repo features, use `"."` as the repository value.

### Part C — Domain Risks Check

If the project has a `spec.md` with a `## Domain Gotchas` section, read it
and incorporate those constraints. If not, ask:

> "Are there any domain-specific risks or unwritten rules that apply to
> this feature area? For example, special libraries that must be used,
> production-only behaviors, or data invariants. If not, say 'no'."

Record any risks provided. These will be stored in plan.json's `knownRisks`
field and passed to every TDD agent as constraints.

### Part D — Scope Lock-Down Questions

After confirming what, why, external context, and domain risks, generate
**5 targeted questions** for the user based on the specific feature
description. These questions must surface decisions and constraints that
**cannot be answered by reading code** — things only the user knows.

Good questions target:
- Scope boundaries ("Should this also handle [adjacent concern], or is
  that out of scope?")
- Backwards compatibility ("Can we change the existing [API/schema/config],
  or must old callers still work?")
- User-facing vs internal ("Is this visible to end users, or purely
  internal infrastructure?")
- Integration expectations ("Should this integrate with [existing system X],
  or stand alone?")
- Quality/completeness bar ("Is this a throwaway prototype or
  production-grade with full test coverage?")

**Do not use generic questions.** Each question must reference specific
aspects of *this* feature. Present all 5 at once for the user to answer
together.

After the user answers, use their responses to generate **10 Exploration
Questions** — specific, codebase-answerable questions that the Explore
agents in Step 2 **MUST answer with `file:line` evidence**. These questions
should force agents to trace the **downstream ripple** of the change, not
just the immediate implementation site. They should connect the feature to
parts of the codebase the agents might otherwise overlook.

Good exploration questions target:
- **Dependency chain**: "Which files import or depend on [module being
  changed]?"
- **Documentation & changelogs**: "Are there changelogs, READMEs, or
  generated docs that track changes like this?"
- **Configuration & registration**: "Is there a config file, route
  registry, or feature flag system where this must be registered?"
- **CI/CD & deployment**: "Are there pipelines, migration scripts, or
  deployment configs affected by this type of change?"
- **Test infrastructure**: "Which existing tests mock or stub the
  components we're modifying?"
- **Similar precedents**: "Has a similar feature been added before? What
  files did *that* change touch?"
- **Cross-cutting concerns**: "Are there logging, metrics, or
  observability hooks that expect to know about this?"
- **Consumer impact**: "What downstream services, scripts, or tools
  consume the output we're changing?"
- **Schema & data**: "Are there migrations, seed data, or fixtures that
  need updating?"
- **Access control & permissions**: "Does the feature area have auth,
  RBAC, or permission checks that need extending?"

Not all categories will apply to every feature. Generate the 10 questions
that are most relevant to *this specific change* based on the user's
answers to the 5 scope questions. Record both question sets and all
answers in research.md (see template).

All gathered context (what, why, external context, domain risks, and
scope lock-down answers) feeds into `research.md` and must be referenced
by the explore agents in Step 2.

**Do not proceed to Step 1 until you have confirmed what, why, external
context, domain risks, and scope lock-down questions.**

---

## Step 1: Read Project Context

Read these files if they exist in the project root (or common locations).
Read them **in full** — do not skim or summarize from headers alone:

- `architecture.md`, `ARCHITECTURE.md`
- `spec.md`, `SPEC.md`
- `CLAUDE.md`
- `README.md`

These provide the architectural and domain context the feature must fit within.

---

## Step 2: Deep Codebase Exploration

Launch Explore agents via the Agent tool to **deeply and in great detail**
understand the codebase areas relevant to this feature. Use the confirmed
feature intent (what and why) and any external context from Step 0 to focus
exploration — the "why" determines which existing patterns and alternatives
are relevant.

**Pass the 10 Exploration Questions** from Part D to every Explore agent.
These questions are the agent's primary research mandate. The agents must:

1. Read the full content of every file identified as relevant — do not
   summarize from file names or directory structure alone
2. **Understand the intricacies** of existing implementations in the
   affected areas — read function bodies, trace data flow, note patterns
3. Trace every caller and downstream consumer of code that will change —
   follow the call graph outward until reaching clear module boundaries
4. Search for existing code that does something similar to what the feature
   requires — note exact `file:line` references
5. Identify naming conventions, test patterns, error handling patterns,
   and architectural boundaries in the affected areas
6. Check `git log` for recent changes in affected areas
7. **Answer every one of the 10 Exploration Questions** with specific
   `file:line` evidence. If an agent cannot answer a question, it must
   explain what it searched and why no answer was found — "not applicable"
   is acceptable only with justification

Each agent reports back structured findings with full file paths and
`file:line` references. Do not accept vague findings like "there's a
utility module" — demand specifics.

**After all agents return**, review their answers to the 10 Exploration
Questions. If any question lacks a `file:line` answer and was not
explicitly marked "not applicable with justification," launch a targeted
follow-up agent to answer that specific question. Do not proceed to
Step 3 with unanswered questions.

---

## Step 3: Answer Three Mandatory Questions

Before writing any plan, answer these questions **in writing** based on
the exploration findings. These prevent the most expensive failure mode:
implementations that work in isolation but ignore existing patterns,
duplicate logic, or violate conventions.

1. **Where does this change belong in the system?**
   Name specific modules, directories, and files. Explain why this location
   and not adjacent alternatives.

2. **Does something similar already exist that must be extended rather
   than duplicated?**
   If yes, cite the exact `file:line` reference. If no, explain what you
   searched and why nothing matched.

3. **Have we solved a related problem before whose pattern can be reused?**
   Cite patterns, utilities, or approaches with `file:line` references.

---

## Step 4: Write research.md

Write all findings to `.claude/features/{slug}/research.md` using the
[research.md template](templates.md#researchmd).

**Populate the Sources Read section.** Before finishing research.md, review
the full conversation history from Steps 0-3 and compile every source
consulted:

- **Files**: Every project file read by you or by explore agents
  (architecture.md, spec.md, CLAUDE.md, README.md, and every source file
  traced during exploration). Use project-relative paths.
- **URLs**: Every URL retrieved via WebFetch during Part B of Step 0. If
  none, write "*No URLs fetched during this research.*"
- **Git History**: Every `git log` or `git diff` command run during
  exploration. Include the exact command and a brief note of what was learned.

Each entry gets a short annotation (a few words) explaining why it was read.
Do not omit sources because they seem unimportant — the section is an audit
trail.

---

## Step 5: Challenge the Request (Devil's Advocate)

Before presenting findings, launch the `devils-advocate` agent via the Agent tool.
Pass it:
- The original feature description
- The path to the research file: `.claude/features/{slug}/research.md`

The agent returns a structured challenge brief covering assumptions,
ambiguities, edge cases, scope creep risks, and a pre-mortem. Insert the
challenge brief into `research.md` under a `## Devil's Advocate Challenge`
heading, **before** the `## Sources Read` section so that Sources Read
remains the final section of the document.

---

## Step 6: Present and Validate

Present the research findings **and** the devil's advocate challenges to
the user together. Highlight:
- Where the change belongs and why
- Existing code to extend or patterns to follow
- Any open questions that need answers before planning
- The key challenges from the devil's advocate — especially assumptions
  that need verification and ambiguities that need clarification

> "Review the research and devil's advocate challenges above. Each challenge
> is marked `[BLOCKING]` or `[ADVISORY]`:
>
> - **[BLOCKING]** items need an individual response: 'acknowledged',
>   'out of scope — reason', or 'disagree — explanation'
> - **[ADVISORY]** items can be acknowledged as a group
>
> You can respond in chat or edit the `> Response:` lines directly in
> `research.md`."

Update `research.md` with the user's responses next to each numbered item.
**Block until all [BLOCKING] items have individual responses.** If the user
provides blanket acknowledgment ("all look fine"), accept it for ADVISORY
items but prompt for specific responses on any BLOCKING items:

> "[N] BLOCKING challenges still need individual responses: items [list].
> These cover [data integrity / security / architectural assumptions] —
> a brief response for each ensures nothing critical is missed."
