---
name: execute
disable-model-invocation: true
argument-hint: "[path to spec file]"
description: >
  Implements a spec produced by /tpe:spec. Reads the spec, creates a
  feature branch, implements the changes, reviews each commit against
  the spec, and self-verifies. Does not push or create PRs.
---

# Execute — Implement a Spec

Read the spec. Do the work. Review against the spec. Verify. Commit.

---

## Input

Resolve `$ARGUMENTS` to a spec file:

1. **Full path** (`docs/specs/features/{slug}/spec.md`): use directly
2. **Slug**: try `docs/specs/features/{slug}/spec.md`
3. **No input**: glob `docs/specs/features/*/spec.md`, present choices

Read the spec in full. This is your contract.

Also read `CLAUDE.md` for project conventions and build/test commands.

---

## Pre-flight

1. **Branch**: if on `main`/`master`, create `feature/{slug}`. If on
   another branch, confirm with user. Never execute on main.

2. **Test baseline**: run the verification command from the spec.
   Record current pass/fail state. If tests already fail, warn the
   user before proceeding.

3. **Validate the spec**: verify that files listed in "Files that
   matter" exist on disk. If any are missing, stop and report.

4. **Run linters and type checkers** if available (check CLAUDE.md
   for commands). Record baseline. These catch mechanical issues
   for free — don't burn AI tokens on what deterministic tools
   handle.

---

## Implementation

Read the spec's Approach, Constraints, Edge cases, and Do NOT
sections. These are your boundaries.

### Principles

- **Follow existing code conventions.** Read the files you're
  modifying before changing them. Match the style, patterns, and
  naming conventions already in the codebase.
- **DRY and YAGNI.** Don't duplicate code that exists. Don't build
  abstractions for hypothetical future needs. Do the simplest thing
  that satisfies the spec.
- **Group related changes into commits that are easy to reverse.**
  One logical change per commit. If the spec involves multiple
  concerns, commit each separately.
- **Use subagents for parallel work.** If the spec involves changes
  to independent files or concerns, dispatch implementation
  subagents in parallel using the Agent tool. Each subagent gets
  the spec + the specific files it should modify. Merge their
  work and verify the combined result.
- **For sequential work, implement directly.** If changes must
  happen in order (e.g., type changes before call site updates),
  implement in the main session without subagents.

### Doing the work

Read each file in "Files that matter" before modifying it. Understand
what's there before changing it.

Implement the changes described in the spec. Use the "Current
behavior" section to find the starting point. Use "Constraints" and
"Edge cases" to guide the implementation. Use "Do NOT" to stay in
scope. Follow the "Approach" — if you encounter a blocker the
approach didn't anticipate, stop and report rather than silently
switching to a different approach.

If the "Alternatives rejected" section lists an approach, do not
use that approach — it was explicitly rejected during planning.

---

## Review Each Commit

**After each logical unit of work, run linters, then dispatch a
review subagent before committing.** Do not self-review — models
fail to correct their own errors 64.5% of the time.

Run linters and type checkers first. Fix any mechanical issues so
the reviewer focuses on logic and spec compliance, not style.

Then launch a review subagent using the Agent tool with this prompt
(fill in the spec path and diff):

```
Read the spec at {spec_path}. Read it in full — it is the ground
truth for this review.

Here is the diff to review:

{paste git diff --cached output}

Check these 5 things against the spec:

1. **Spec compliance**: does the change match the Approach? Does
   it violate any Constraints or Do NOT rules? Were any
   "Alternatives rejected" approaches accidentally used?
2. **Edge case coverage**: for each edge case in the spec, is it
   handled in the code?
3. **Error handling**: are error paths covered? AI-generated code
   has 2x more error handling gaps — look specifically for missing
   error paths.
4. **Regression risk**: were files outside "Files that matter"
   modified without justification?
5. **Logic errors**: trace data flow through the changed code.
   Off-by-one, null handling, race conditions.

Do NOT review style, formatting, or naming — linters handle that.

For every finding, include:
- file:line location
- Trace: the data flow or logic path
- Severity: BLOCKING (will break production) or SHOULD_FIX
- Fix: what specifically to change

If no issues found, return PASS. Do not manufacture findings.
False positives erode trust faster than false negatives.
```

If the subagent returns findings, fix them before committing. If
PASS, commit with a clear message.

Keep commits under 400 changed lines — review effectiveness drops
sharply beyond that threshold. If a logical change exceeds 400
lines, split it into smaller commits.

---

## Verification

**You are not done until the spec's verification criteria pass.**

Run the verification command from the spec. Check every assertion:
- All existing tests listed must still pass
- All new behaviors described must be verified
- If any check fails, fix it and re-verify

Run linters and type checkers again. Fix any new violations
introduced by your changes.

Do not commit until verification passes. Do not rely on your own
judgment that the code is correct — run the actual commands.

If verification fails after five fix attempts, stop and report the
failure to the user with details. Do not keep retrying in a loop.

---

## Finalize

After verification passes:

1. Run the full test suite (from CLAUDE.md's operational commands)
   to check for regressions beyond the spec's verification scope.
   Fix any regressions before finalizing.

2. Update the spec's "Last updated" date if implementation revealed
   new constraints or edge cases worth recording.

3. Flip the spec's `Status` from `Waiting Implementation` to
   `Implemented` — both in the spec file's header and in its row
   in the Spec Index (`docs/specs/spec.md`). If the spec lives
   under `docs/specs/bugs/`, update the Bugs table; otherwise
   update the Features table.

4. Present a summary:
   - **Branch**: name and commit count
   - **What was done**: 2-3 sentences
   - **Verification**: pass/fail status
   - **Review findings**: any issues caught and fixed during
     per-commit review
   - **Needs attention** (if any): anything unexpected, deviations
     from the spec, or issues the user should review

**Do NOT push or create a PR.** The user handles this.
