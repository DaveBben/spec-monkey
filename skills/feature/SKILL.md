---
name: feature
effort: high
model: opus
disable-model-invocation: true
argument-hint: "[feature description or change request]"
allowed-tools:
  - Agent
  - Skill
  - TodoWrite
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - AskUserQuestion
  - LSP
description: >
  Standalone feature planning skill. Takes a free-form feature or change
  request, deeply researches the codebase, produces an impact map and
  implementation plan with real code snippets, then decomposes into tasks
  compatible with /execute. Human reviews via annotation cycle at each phase.
  Do NOT use for bugs (use /bug + /triage).
  Do NOT use for trivial one-line changes (just make the change).
---

# Feature Planner

> Takes a feature description or change request, deeply researches the
> codebase, produces an impact map and plan with actual code snippets, then
> decomposes into implementation tasks. Human reviews at each phase via an
> annotation cycle. Expect 15-30 minutes.

```
User describes feature
    |
/feature (this skill)
    |
    |-- Phase 0: Deep research → research.md
    |-- Phase 1: Impact map → impact-map.md
    |-- Phase 2: Plan with code snippets → plan.md
    |-- Phase 3: Annotation cycle (1-6 rounds)
    |-- Phase 4: Task breakdown → tasks.md + task JSON files
    |
    v
/execute .claude/features/{slug}/
    or
User runs implementation command from tasks.md
```

All artifacts are written to `.claude/features/{slug}/` in the project root.
Task JSON files live under `tasks/` within that directory.

### Supporting Files

- [Templates](references/templates.md) — Markdown and JSON templates for all artifacts
- [Implementation Guidance](references/implementation-guidance.md) — Reference for effective execution

### /execute Compatibility

This skill writes task JSON files to `.claude/features/{slug}/tasks/` so
`/execute` can pick them up. **Note:** `/execute` currently resolves tasks
via story IDs from the global artifact store at
`~/.claude/backlog-driven-development/artifacts/`. To use `/execute` with
`/feature` output, `/execute` must be updated to accept a feature directory
path as an alternative input. Until then, use the implementation command
from the bottom of `tasks.md` instead.

---

## Input Handling

If `$ARGUMENTS` were provided, classify them:

1. **Existing feature directory** (path to a `.claude/features/{slug}/` directory):
   Enter **Resumption Mode** — check which artifacts exist and resume from the
   last incomplete phase:
   - No `research.md` → start at Phase 0
   - Has `research.md` but no `impact-map.md` → start at Phase 1
   - Has `impact-map.md` but no `plan.md` → start at Phase 2
   - Has `plan.md` with `Status: Draft` → start at Phase 3 (annotation cycle)
   - Has `plan.md` with `Status: Approved` but no `tasks.md` → start at Phase 4
   - Has `tasks.md` → tell the user: "This feature is fully planned. Use the
     implementation command at the bottom of tasks.md to implement."

2. **Free-form text description**: Use as the feature description. Generate a
   URL-safe slug from the first 3-5 significant words (e.g., "add webhook retry
   with exponential backoff" becomes `add-webhook-retry-exponential`). If
   `.claude/features/{slug}/` already exists, append a short numeric suffix
   (`-2`, `-3`). Create the directory and proceed to Phase 0.

3. **No input**: Ask the user to describe the feature or change they want:
   > "Describe the feature or change you want to implement. Be as specific as
   > you like — I'll research the codebase and ask clarifying questions."

---

## Phase 0: Deep Research (Pre-flight)

**This phase must complete before any planning begins.** The research artifact
is your review surface. If the research is wrong, the plan will be wrong, and
the implementation will be wrong. Correcting a wrong assumption here costs
nothing. Catching it in a pull request costs hours.

### Step 1: Read Project Context

Read these files if they exist in the project root (or common locations).
Read them **in full** — do not skim or summarize from headers alone:

- `architecture.md`, `ARCHITECTURE.md`
- `spec.md`, `SPEC.md`
- `CLAUDE.md`
- `README.md`

These provide the architectural and domain context the feature must fit within.

### Step 2: Deep Codebase Exploration

Launch Explore agents via the Agent tool to **deeply and in great detail**
understand the codebase areas relevant to this feature. The agents must:

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

Each agent reports back structured findings with full file paths and
`file:line` references. Do not accept vague findings like "there's a
utility module" — demand specifics.

### Step 3: Answer Three Mandatory Questions

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

### Step 4: Write research.md

Write all findings to `.claude/features/{slug}/research.md` using the
[research.md template](references/templates.md#researchmd).

### Step 5: Present and Validate

Present the research findings to the user. Highlight:
- Where the change belongs and why
- Existing code to extend or patterns to follow
- Any open questions that need answers before planning

Ask the user to confirm or correct the findings before proceeding.
**Do not proceed to Phase 1 until the user acknowledges the research.**

---

## Phase 1: Impact Map

Before individual tasks are written, scan the actual codebase and produce a
repository impact map — which files, which symbols, which patterns are
affected. The user reviews this map before anything else happens.

**The failure mode this catches:** structurally incorrect plans that produce
code that compiles but solves the wrong problem — wrong module, missed
dependency, nonexistent endpoint pattern.

### Step 1: Build the Impact Map

Using the research from Phase 0, identify every file and symbol that will
be created, modified, or affected by this feature. For each entry, note the
action (create/modify/delete) and the specific symbols involved.

### Step 2: Write impact-map.md

Write to `.claude/features/{slug}/impact-map.md` using the
[impact-map.md template](references/templates.md#impact-mapmd).

### Step 3: Present for Review

Present the impact map to the user. Ask:

> "Review the impact map above. Does it capture all the files and areas
> that need to change? Is anything missing or incorrectly scoped?"

**Do not proceed to Phase 2 until the user acknowledges the impact map.**

---

## Phase 2: Plan Document

One plan file per feature. The plan holds the full intent, architectural
decisions, code snippets showing **actual changes** (not pseudocode, not
descriptions), file paths, and trade-offs.

### Step 1: Write plan.md

Write to `.claude/features/{slug}/plan.md` using the
[plan.md template](references/templates.md#planmd).

**Include actual code snippets.** Not pseudocode, not descriptions — the
actual proposed signatures, schema changes, and data shapes. When a reference
implementation exists in the codebase, paste the relevant section alongside
the proposed change. When the AI has a concrete reference, it implements
against that reference instead of designing from scratch.

**Explicitly forbid implementation.** The plan ends with "Do not implement
yet." Without this, the AI will jump to code the moment it thinks the plan
is good enough.

### Step 2: Present the Plan

Present the plan to the user and transition to the annotation cycle:

> "The plan is at `.claude/features/{slug}/plan.md`. You can:
>
> 1. **Annotate in your editor** — open the file, add inline notes starting
>    with `> NOTE:`, `> FIX:`, or `> REMOVE:` at the exact location of each
>    issue, then tell me 'review'
> 2. **Give feedback here** — describe corrections in chat and I'll update
>    the plan
> 3. **Approve** — say 'approve' if the plan is ready for task breakdown"

---

## Phase 3: Annotation Cycle

This is the highest-leverage step. The AI writes code-level detail. The
user injects judgment — product priorities, business constraints, and
engineering trade-offs the AI cannot know.

### How Inline Annotations Work

The user opens `plan.md` in their editor and adds notes directly into the
document at the exact location of each issue:

- `> NOTE: [correction or context]` — domain knowledge the AI lacks
- `> FIX: [what's wrong and what it should be]` — correcting an assumption
- `> REMOVE: [reason]` — rejecting a proposed approach

Real annotation examples:
- `> FIX: use drizzle:generate for migrations, not raw SQL`
- `> FIX: this should be a PATCH not a PUT`
- `> REMOVE: we don't need caching here`
- `> NOTE: the queue consumer already handles retries, this retry logic is redundant`
- `> FIX: the visibility field needs to be on the list, not individual items — restructure the schema section`

### Processing Annotations

When the user says "review" (or provides chat feedback):

1. Read the updated `.claude/features/{slug}/plan.md`
2. Find all annotation markers — match `> NOTE:`, `> FIX:`, `> REMOVE:`
   case-insensitively with flexible whitespace
3. Address **every** annotation — do not skip any
4. Remove the annotation markers from the updated plan
5. Write the updated plan back to `plan.md`
6. Present what changed in response to each annotation

### Cycle Repetition

After addressing annotations, ask:

> "I've addressed all your annotations. Review the updated plan — add more
> notes or say 'approve' when satisfied."

Repeat up to 6 annotation cycles. If the plan is still not approved after
6 cycles, ask the user if the feature scope needs to be reconsidered.

### On Approval

When the user says "approve":

1. Update the plan's `**Status**` to `Approved`
2. Write the updated `plan.md`
3. Proceed to Phase 4

---

## Phase 4: Task Breakdown

After the plan is approved, produce a granular task list as a separate
artifact. The plan carries the long-horizon intent; each task stays within
a bounded working set.

### Task Granularity Rules

- **Count decisions, not lines.** A task that touches 200 lines but makes
  one decision is better than a 30-line task requiring three design choices.
- **Single-function scope.** Single-function tasks achieve ~87% AI accuracy.
  Multi-file tasks achieve ~19%. Keep tasks within one file or one function.
- **The right size** is the smallest coherent unit that preserves
  architectural clarity — not so granular it fragments the mental model,
  not so broad it requires design choices during execution.
- **Size warning.** If the task list exceeds ~50 tasks, the feature is
  scoped too broadly. Break it into smaller independently-deliverable changes.

### Task Format

Each task must contain:

- **The single file** to touch (explicit path, not a guess)
- **The specific function or symbol** to create or modify
- **A reference** to an existing pattern to follow (exact function name
  and `file:line`)
- **What NOT to do** — adjacent files or scope that must not change
- **A "done when" condition** that can be checked mechanically

Use **"must"** throughout task language. "Should" significantly reduces AI
adherence. "Must" aligns with technical standards language and the AI treats
it as a hard constraint.

### Step 1: Write tasks.md

Write to `.claude/features/{slug}/tasks.md` using the
[tasks.md template](references/templates.md#tasksmd).

### Step 2: Write Task JSON Files

Write each task as a JSON file to `.claude/features/{slug}/tasks/` using
the [task JSON schemas](references/templates.md#task-json-schemas):

- `tasks/plan.json` — plan-level information
- `tasks/task_{N}.json` — one per task (e.g. `task_0.json`, `task_1.json`)

### Step 3: Present Tasks for Review

Present the task breakdown to the user:

> "Task breakdown complete — {N} tasks in `.claude/features/{slug}/tasks.md`.
>
> Review the tasks. You can request changes or approve. When ready to
> implement, use the implementation command at the bottom of tasks.md."

---

## Common Failure Patterns

- **Skimming instead of reading** — Phase 0 exists because the AI reads at
  signature level and moves on. Force deep reading with specific instructions:
  "read the full function body", "trace every caller", "understand the intricacies."
- **Planning before researching** — The research artifact must be written and
  reviewed BEFORE any plan. Without it, plans are structurally incorrect.
- **Pseudocode in the plan** — The plan must contain actual code snippets.
  Pseudocode leads to implementations that diverge from intent.
- **Accepting vague annotations** — If the user annotates "make this better",
  push back: "Better how? Faster? More readable? Different API shape?"
- **Skipping the impact map review** — The impact map catches wrong-module
  errors. Skipping it means discovering structural problems after tasks are
  written.
- **Tasks with multiple files** — Each task must target one file. Multi-file
  tasks drop AI accuracy from ~87% to ~19%.
- **"Should" instead of "must"** — "Should" reduces AI adherence. Use "must"
  for all task constraints.
- **Missing "Do NOT" boundaries** — Without explicit boundaries, tasks expand
  into adjacent work. Every task needs a "Do NOT" field.
- **Feature too large** — More than ~50 tasks means the feature needs to be
  split. Plan at the right altitude.
