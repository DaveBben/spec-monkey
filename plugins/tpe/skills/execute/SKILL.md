---
name: execute
disable-model-invocation: true
effort: max
argument-hint: "[feature or bug directory path or slug]"
model: opus
description: >
  Implements approved tasks from /tpe:plan or /tpe:bug. For each task:
  dispatches a single implementation agent.
  After all tasks: full test suite, finalize. Does not
  commit, push, or create PRs.
---

# Plan Executor

Execute approved tasks. For each task: implement.
After all tasks: full tests → finalize.

### Supporting Files

- [PR Templates](references/pr-templates.md) — Feature and bugfix PR body templates
- [Resumption](references/resumption.md) — Checkpoint format and session resumption
- [Error Handling](references/error-handling.md) — Plan deviation responses
- [Design Decisions](references/design-decisions.md) — Rationale for single-agent and no-TDD choices

## Execution Model

```
Main session:
  ├── Input resolution → load plan.json + task JSONs
  ├── Pre-flight (branch, validate, test baseline)
  ├── For each task (sequential):
  │   ├── Pre-task check
  │   ├── Implementation agent (single agent — reads task, implements, verifies)
  │   └── Update task status
  ├── Full test suite + lint
  └── Finalize (user handles push + PR)
```

---

## Input Resolution

Resolve `$ARGUMENTS`:

1. **Feature/bug directory path**: Verify `tasks/plan.json` exists.
   If missing and path is under `.claude/bugs/`:
   > "No tasks found at `.claude/bugs/{slug}/tasks/`. Run `/tpe:bug {slug}` to complete investigation and produce task JSONs."
   If missing and path is under `.claude/features/`:
   > "No plan.json found at `.claude/features/{slug}/tasks/`. Run `/tpe:plan {slug}` to produce the plan and task JSONs."
2. **Slug**: Try `.claude/features/{slug}/` then `.claude/bugs/{slug}/`.
3. **No input**: Glob for available plans, present choices.

Determine source type from path (`feature` or `bugfix`) for branch naming.

Load `plan.json` + all `task_*.json` files sorted by numeric ID. If `execution-state.json` exists, follow [resumption procedure](references/resumption.md).

---

## Pre-flight

### Branch Safety (hard stop)

If on `main`/`master`: create `feature/{slug}` or `bugfix/{slug}`. If on another branch: confirm with user. Never execute on main.

### Validate Task JSONs

For each task:
- `files` paths exist (modify) or don't exist (create)
- `relevantFiles` paths with `action: modify` exist on disk
- `relevantFiles` paths with `action: create` do NOT exist on disk
- `doNot`, `acceptanceCriteria` non-empty; `doneWhen` non-empty
- `verificationCommand` is syntactically valid

**Critical** (report and ask — do not proceed):
- File with `action: modify` does not exist on disk
- File with `action: create` already exists (collision)
- `acceptanceCriteria` or `doneWhen` is empty (task is unimplementable)

**Minor** (warn and continue):
- `verificationCommand` references a binary not on PATH (may be in devDependencies — try running it first)
- File exists but has been modified since planning (content differs from plan.md snippets) — note the drift, proceed unless the changed symbol is in the task's `dependencyChain`
- `relevantFiles` path differs in casing or trailing slash

### Test Baseline

Run full test suite. Record passing count and known failures.

### Spec Freshness

If any `spec.md` (root or domain) has `Last verified` >30 days old and plan date >7 days old: warn and continue. Skip if plan was just produced.

### Brainstorm.md Check

If any task qualifies as Complex tier, verify `brainstorm.md` exists. If missing: warn (advisory — continue), offer to downgrade to Standard tier or return to `/tpe:think`.

### Gate Check

- `plan.json` status is `Approved`
- Read `constraints` and `criticalReminders` — passed to every agent

---

## Fast Path (Small Plans)

If the plan has **≤2 tasks**, total estimated size is **under 200 lines**, the combined unique `files` across all tasks is **≤4**, and **no task would be classified as Complex tier**, skip task-by-task execution:

1. Dispatch a single implementation agent with each task JSON path (not merged), plan constraints, and the union of atRiskTests
2. If agent returns DONE and all regressionChecks pass: proceed to Post-Implementation
3. If agent STOPs or exceeds 200 lines: fall back to standard task-by-task execution below

≤2 tasks / ≤4 files keeps the agent within the context interference threshold. Do not merge task JSONs into a single blob — pass them as separate files so the agent reads each task's boundaries independently.

---

## Task Execution

Create TodoWrite entries for all tasks for user visibility.

### Slice-Aware Execution

If `totalSlices > 1`: process sequentially. Each slice = branch → tasks → post-implementation → finalize. Slice 1 branches from main; subsequent slices branch from previous (stacked PRs).

### Complexity Tier

Before dispatching each task, classify it:

| Tier | Criteria | Agent Config |
|------|----------|-------------|
| **Simple** | 1 file, clear reference, no interface changes | maxTurns: 20, model: haiku |
| **Standard** | 2-3 files, no shared interface/type changes | maxTurns: 50, model: sonnet (default) |
| **Complex** | 4 files, OR any task that changes a shared interface/type (regardless of file count) | maxTurns: 75, model: opus, also pass `brainstorm.md` path for additional approach context |

**Tiebreaker:** if a task matches multiple tiers, always promote to the higher tier. A 2-file task that changes a shared interface is Complex, not Standard — the interface change is the risk driver, not the file count.

If a Simple task agent STOPs or hits maxTurns (agent-level failure), retry once at Standard tier before marking BLOCKED. This tier promotion applies only to agent-level failures — if the agent returns DONE but regressionCheck fails, retry at the same tier with the failure details (see Step 2 verification below).

### For Each Task

#### Step 1: Pre-task Check

- Do `relevantFiles` paths match expected state (exist/not-exist)?
- Is `git status` clean? (Must be — prior task was committed in Step 3.) If not clean, something went wrong. Pause (blocking — wait for user response) before proceeding.
- If `blockedBy` lists tasks that aren't DONE, skip and report.
- If `execution-state.json`'s `executionNotes` has a BLOCKED note for this task, show it to the user and ask whether to proceed.

If assumptions violated, ask the user.

#### Step 2: Implement

**Always dispatch — never implement in-line.** Even if the task looks like a one-line change you could make faster yourself, dispatch the agent. Your orchestrator context accumulates noise from prior tasks, plan JSONs, test output, and TodoWrite churn; the agent gets a clean read of just this task. Skipping dispatch for "small" tasks is the failure mode this skill is designed to prevent. For trivially small tasks, downshift to `model: haiku` (Simple tier) rather than implementing directly.

Dispatch the `code-implementor` agent using the Agent tool with `subagent_type: "code-implementor"`. Override `maxTurns` and `model` to match the task's complexity tier. Pass these items in this order (the agent reads top-down, so frontload what constrains behavior):

1. **Plan constraints** verbatim — global boundaries the agent must not violate
2. **Task JSON path** — the full spec (acceptanceCriteria, reference, files, doNot, dependencyChain, etc.)
3. **Plan JSON path** — for broader context
4. **Test baseline** — known failures to ignore
5. **Instruction**: "Re-read `doNot` and plan constraints before your verification step" (counters instruction fade-out)

The agent reads `doNot` and `dependencyChain` from the task JSON directly — do not duplicate them verbatim in the prompt.

If STOPPED: mark `BLOCKED`, record in `executionNotes[task_id]`, ask user.

**Trust-but-verify**: run `regressionCheck` from the orchestrator (do not rely on agent self-report). Check three things:

1. **Exit code** is non-zero → fail
2. **Scan output** for "fail", "error", "FAIL", "ERROR" → fail
3. **At-risk test coverage**: for each test in the task's `atRiskTests`, verify its name or file appears in the test output. If a test is absent from output entirely (not failed — *absent*), it was skipped or not collected. This is a failure — a skipped at-risk test provides no regression signal. Report which tests were missing.

If any check fails: one retry, then `BLOCKED`. On the retry, include the specific failure in the agent prompt ("regressionCheck failed: [reason]") so it can target the fix.

#### Step 3: Commit

Commit the agent's uncommitted changes with message `task_{N}: {task title}`. Do not push.

#### Step 4: Handoff Check

If any pending task has this task's ID in its `blockedBy`, dispatch the `task-handoff-checker` agent (completed + dependent task JSON paths, known-failures baseline). Returns `PASS` or `BLOCKED — [reason]`.

On BLOCKED: record in `executionNotes[dependent_task_id]`, warn user. Skip entirely if no task depends on this one.

#### Step 5: Size & Drift

- **Task >200 lines**: note in `executionNotes`, inform user. If remaining tasks depend on this one, continue but flag the size for review. If remaining tasks are independent, pause for user review before continuing (blocking — wait for response).
- **Cumulative slice >500 lines**: warn and pause (blocking — wait for response). Ask to continue or split. If splitting, finalize the current slice (post-implementation, tests, commit) before starting the next.
- **Agent used >80% maxTurns**: flag possible drift. If the task is DONE despite high turn count, note it but continue. If the next task shares any entry in its `files` array with this task, promote it one complexity tier (the high turn count suggests the area is harder than estimated).

#### Step 6: Update Status & Mask

Set task `DONE`/`BLOCKED`. Update `execution-state.json` with one-line summary. For next task prompt: reference only summaries for prior tasks (keep last 2 full results for continuity).

### Context Pause Gate

Every **6 completed tasks**: pause, report progress, ask user to check `/context` or start fresh session (resumption picks up automatically).

### Blocked Tasks

Record reason, set `BLOCKED`, check dependents, ask user to continue or stop.

---

## Post-Implementation & Finalize

After all tasks in the current slice:

1. Run full test suite + lint. Fix failures.

2. **Refactor pass (advisory + gated).** Dispatch the `refactor-opportunities` agent with the feature directory path and branch base (`git merge-base main HEAD` for slice 1, or the previous slice's tip for stacked slices). If it returns `No refactor opportunities identified.`, skip.

   Otherwise present the numbered list verbatim and prompt:
   > "Select refactors to apply. Reply `all`, `none`, a category (`consistency` / `complexity` / `comments`), or numbers (e.g. `1,3,5-6`)."

   On `none`, skip. Otherwise apply the selected edits with Edit/Write at the agent's `file:line` anchors, re-run tests + lint (same fix-until-green pattern as step 1), and commit as a single `refactor: post-task cleanup` — new commit, do not amend task commits.

3. If `spec.md` exists and capabilities changed, update it.
4. Set plan.json status to `COMPLETE`.
5. Present summary structured as:
   - **Branch**: name and commit count
   - **Tasks**: completed / blocked / total, with one-line status per task
   - **Test results**: pass count vs baseline, any new failures
   - **Flags**: any executionNotes entries (size warnings, drift flags, handoff issues)
   - **Needs attention** (if any): blocked tasks, unresolved issues, or deviations from the plan that the user should review before proceeding

   Then say:
   > "Clear your context and run `/tpe:review` to review the changes."
6. Point to [PR templates](references/pr-templates.md) for body content.

**Do NOT push or create a PR.** User handles this manually.
See [error handling](references/error-handling.md) for plan deviations.
