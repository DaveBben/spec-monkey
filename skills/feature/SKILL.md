---
name: feature
effort: high
model: opus
disable-model-invocation: true
argument-hint: "[feature description or change request]"
description: >
  Standalone feature planning skill. Takes a free-form feature or change
  request, deeply researches the codebase, produces an impact map and
  implementation plan with real code snippets, then decomposes into tasks
  compatible with /cks:execute. Human reviews via annotation cycle at each phase.
  Do NOT use for bugs (use /cks:bug).
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
/cks:feature (this skill)
    |
    |-- Phase 0: Clarify intent → deep research → research.md
    |-- Phase 1: Impact map → impact-map.md
    |-- Phase 2: Plan with code snippets → plan.md
    |-- Phase 3: Annotation cycle (1-6 rounds)
    |-- Phase 4: Task breakdown → tasks.md + task JSON files
    |
    v
/cks:execute .claude/features/{slug}
```

All artifacts are written to `.claude/features/{slug}/` in the project root.
Task JSON files live under `tasks/` within that directory.

### Supporting Files

- [Templates](references/templates.md) — Markdown and JSON templates for all artifacts
- [Common Failure Patterns](references/failure-patterns.md) — Known pitfalls and how to avoid them

### /cks:execute Compatibility

This skill writes task JSON files to `.claude/features/{slug}/tasks/` so
`/cks:execute` can pick them up. Pass the feature directory path to `/cks:execute`:

```
/cks:execute .claude/features/{slug}
```

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
   - Has `tasks.md` → tell the user: "This feature is fully planned. Run
     `/cks:execute .claude/features/{slug}` to implement."

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

Follow the [Research Protocol](references/research-protocol.md) step by step:

1. **Step 0 — Clarify Intent and Gather Context**: Confirm what and why
   (Part A), check for external context (Part B), surface domain risks
   (Part C), then run the Scope Lock-Down (Part D) — 5 user questions to
   lock scope, then generate 10 Exploration Questions that force agents to
   trace the downstream ripple of the change
2. **Step 1 — Read Project Context**: Read architecture.md, spec.md,
   CLAUDE.md, README.md in full
3. **Step 2 — Deep Codebase Exploration**: Launch Explore agents with
   the 10 Exploration Questions as their primary research mandate. Every
   question must be answered with `file:line` evidence or marked "not
   applicable" with justification. Follow up on unanswered questions
   before proceeding
4. **Step 3 — Three Mandatory Questions**: Where does this belong? Does
   something similar exist? Have we solved a related problem before?
5. **Step 4 — Write research.md**: Compile all findings using the
   [research.md template](references/templates.md#researchmd), including
   both question sets and all answers in the Scope Lock-Down section
6. **Step 5 — Devil's Advocate**: Launch the `devils-advocate` agent to
   challenge the research before the user sees it
7. **Step 6 — Present and Validate**: Present findings + challenges to
   the user. Block until all `[BLOCKING]` items have individual responses

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

Present the impact map to the user:

> "Review the impact map above. If any files or areas are missing or
> incorrectly scoped, let me know. Otherwise, I'll continue to the plan."

If the user provides corrections, update the impact map. If the user
confirms or does not object, proceed to Phase 2. **Only block if the
impact map is clearly incomplete** (e.g., missing an entire module the
feature depends on).

---

## Phase 2: Plan Document

One plan file per feature. The plan holds the full intent, architectural
decisions, code snippets showing **actual changes** (not pseudocode, not
descriptions), file paths, and trade-offs.

### Step 1: Determine Delivery Strategy

Before writing the plan, estimate total change size from the impact map. Use
line counts from similar existing files, or rough estimates based on the
number of functions and types being created or modified.

- **Under ~500 lines total**: Single slice (one PR). Note "Single PR" in the
  Delivery Strategy section and proceed.
- **Over ~500 lines total**: Design the plan around multiple **vertical
  slices**. Each slice is one complete thin path through the system — database
  change + service logic + API endpoint + test for *one specific capability*.
  Never slice horizontally (all DB changes, then all service changes). Each
  slice must leave the project in a working, testable state when merged
  independently.

Document the delivery strategy in the plan using the
[Delivery Strategy template](references/templates.md#planmd).

### Step 2: Write plan.md

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

### Step 3: Verify the Plan Against the Codebase

Before presenting the plan to the user, launch the `plan-verifier` agent
via the Agent tool. Pass it the path to `.claude/features/{slug}/plan.md`.

The agent checks that:
- Every `file:line` reference in the plan still matches the codebase
- Types, interfaces, and function signatures referenced actually exist
- Assumptions listed in the `## Assumptions` section are true
- Pattern references haven't been recently changed

If the verifier returns discrepancies:
1. Apply the `@FIX:` annotations it provides to the plan
2. Address each fix (update references, correct assumptions)
3. Write the corrected plan back to `plan.md`

This catches "looks right but is subtly wrong" errors before the user
sees the plan — the most expensive class of mistakes in brownfield work.

### Step 4: Present the Plan

Present the plan to the user and transition to the annotation cycle:

> "The plan is at `.claude/features/{slug}/plan.md`. You can:
>
> 1. **Annotate in your editor** — open the file, add inline notes starting
>    with `@NOTE:`, `@FIX:`, or `@REMOVE:` at the exact location of each
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

- `@NOTE: [correction or context]` — domain knowledge the AI lacks
- `@FIX: [what's wrong and what it should be]` — correcting an assumption
- `@REMOVE: [reason]` — rejecting a proposed approach

The `@` prefix avoids collisions with markdown blockquotes (`>`).

Real annotation examples:
- `@FIX: use drizzle:generate for migrations, not raw SQL`
- `@FIX: this should be a PATCH not a PUT`
- `@REMOVE: we don't need caching here`
- `@NOTE: the queue consumer already handles retries, this retry logic is redundant`
- `@FIX: the visibility field needs to be on the list, not individual items — restructure the schema section`

### Processing Annotations

When the user says "review" (or provides chat feedback):

1. Read the updated `.claude/features/{slug}/plan.md`
2. Find all annotation markers — match `@NOTE:`, `@FIX:`, `@REMOVE:`
   case-insensitively with flexible whitespace
3. Address **every** annotation — do not skip any
4. Remove the annotation markers from the updated plan
5. Write the updated plan back to `plan.md`
6. Present what changed in response to each annotation

### Cycle Counter

Track the annotation cycle count with an HTML comment in `plan.md`:

```
<!-- annotation-cycle: 1/6 -->
```

Insert this comment after the `**Status**` line when entering Phase 3.
Increment the counter each time annotations are processed. Read it on
each cycle to determine the current count — do not rely on session memory.

### Cycle Repetition

After addressing annotations, increment the cycle counter and ask:

> "I've addressed all your annotations (cycle {N}/6). Review the updated
> plan — add more notes or say 'approve' when satisfied."

If the plan is still not approved after 6 cycles, ask the user if the
feature scope needs to be reconsidered.

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
- **Single-concern scope.** Each task must target one logical concern,
  typically touching 1-3 files. An endpoint task that touches route +
  controller + types is one concern. A task that touches auth middleware +
  database schema + frontend component is three concerns.
- **The right size** is the smallest coherent unit that preserves
  architectural clarity — not so granular it fragments the mental model,
  not so broad it requires design choices during execution.
- **PR size ceiling.** Larger PRs receive less thorough review — keep PRs
  focused. Target under 200 lines per slice for genuine reviewer attention.
  If the task list for a single slice implies >500 lines of changes, split
  the slice further.
- **Size warning.** If the task list exceeds ~50 tasks, the feature is
  scoped too broadly. Split into multiple slices or reduce scope. If a single
  slice exceeds ~15 tasks, it is likely too large for one PR.

### Vertical Slices, Not Horizontal Layers

If the plan's Delivery Strategy specifies multiple slices, assign every task
to a slice. Tasks within a slice must form a complete vertical path — never
group by layer (all schema tasks, then all service tasks, then all API tasks).

Each slice must satisfy all four of these questions:
1. **Can a reviewer review this slice in 15 minutes?** If not, it's too large.
2. **Is there exactly one reason this slice would be reverted?** If it could
   be reverted for unrelated reasons, it combines separate concerns.
3. **Can you list every file it touches right now?** If not, the scope is
   unclear.
4. **Is the test for each task obvious before the code exists?** If not, the
   task is underspecified.

If any answer is "no," re-slice.

**One-sentence test:** Describe each slice's diff in one sentence. The sentence
must describe the *change* ("Add retry column to webhooks table and expose
retry count in the status endpoint"), not the *goal* ("Implement webhook
retries"). If you can't write that sentence, the slice is too big or too vague.

### Task Format

Each task must contain:

- **The file(s)** to touch (explicit paths, typically 1-3 files for one concern)
- **The specific function or symbol** to create or modify
- **A reference** to an existing pattern to follow (exact function name
  and `file:line`)
- **What NOT to do** — adjacent files or scope that must not change
- **A "done when" condition** that can be checked mechanically

Use **RFC 2119 keywords** to communicate constraint levels clearly:
- **MUST** (uppercase) = absolute requirement, pair with a verification step
- **SHOULD** = strong preference, deviation acceptable with justification
- **MAY** = full discretion

LLMs have learned these semantic distinctions from RLHF training data that
includes RFC-style documents. Using them consistently establishes a clear
contract between the plan and the executing agents.

### Step 1: Write tasks.md

Write to `.claude/features/{slug}/tasks.md` using the
[tasks.md template](references/templates.md#tasksmd).

### Step 2: Write Task JSON Files

Write each task as a JSON file to `.claude/features/{slug}/tasks/` using
the [task JSON schemas](references/templates.md#task-json-schemas):

- `tasks/plan.json` — plan-level information
- `tasks/task_{N}.json` — one per task (e.g. `task_0.json`, `task_1.json`)

### Step 3: Validate Task JSONs

After writing all task JSONs, validate each one:

1. **File paths**: every path in `files`, `testContext`, and
   `implementationContext` either exists on disk or has a matching
   `relevantFiles` entry with `action: "create"`
2. **Required fields non-empty**: `doNot` has at least one entry,
   `acceptanceCriteria` has at least one entry, `doneWhen` is non-empty
3. **Context separation**: no path appears in both `testContext` and
   `implementationContext` (they serve different agents)
4. **Verification command**: `verificationCommand` is syntactically valid
   (contains a recognizable test runner command)
5. **Environment check**: if `environmentCheck` is set, verify the command
   is syntactically valid

Fix any violations before presenting to the user. If a path doesn't exist
and isn't marked as `create`, flag it as a planning error and correct the
task or ask the user.

### Step 4: Present Tasks for Review

Present the task breakdown to the user:

> "Task breakdown complete — {N} tasks in `.claude/features/{slug}/tasks.md`.
>
> Review the tasks. You can request changes or approve. When ready to
> implement, run:
>
> `/cks:execute .claude/features/{slug}`"

---

## Common Failure Patterns

See [references/failure-patterns.md](references/failure-patterns.md) for the full list of known pitfalls.
