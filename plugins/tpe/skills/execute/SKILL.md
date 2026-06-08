---
name: execute
disable-model-invocation: true
model: opus
effort: high
argument-hint: "[path to spec file]"
tools:
  - Read
  - Glob
  - Grep
  - Bash
description: >
  Implements a spec produced by /tpe:spec. Reads the spec, creates a
  feature branch, implements the changes, self-verifies, then runs
  parallel staff and QA reviews. Does not push or create PRs.
---

# Execute — Implement a Spec

Read the spec. Do the work. Review against the spec. Verify. Commit.

---

## Input

Resolve `$ARGUMENTS` to a spec file:

1. **Full path** (`docs/specs/features/{slug}/spec.md`): use directly
2. **Slug**: try `docs/specs/features/{slug}/spec.md`
3. **No input**: glob `docs/specs/features/*/spec.md` and
   `docs/specs/bugs/*/spec.md`. Read each spec's YAML frontmatter and
   filter to those with `Status: Waiting Implementation`. Present the
   matching specs as a numbered menu showing slug and title, then ask
   the user to choose. If none are waiting, tell the user.

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
   for free.

---

## Implementation

Read the spec's Approach, Constraints, Edge cases, and Do NOT
sections. These are your boundaries.

### Principles

- **Follow existing code conventions.** Read the files you're
  modifying before changing them. Match the style, patterns, and
  naming conventions already in the codebase.
- **DRY and YAGNI.** Reuse existing logic rather than reimplementing
  it. Don't build abstractions for hypothetical future needs — prefer
  a little repetition over the wrong abstraction. Do the simplest
  thing that satisfies the spec.
- **Group related changes into commits that are easy to reverse.**
  One logical change per commit. If the spec involves multiple
  concerns, commit each separately.
- **Use subagents for parallel work.** If the spec involves changes
  to independent files or concerns, dispatch implementation
  subagents in parallel using the Agent tool. Each subagent gets
  the spec + the specific files it should modify. Merge their
  work and verify the combined result.
- **Prefer Haiku agents for simple file edits.** For mechanical
  changes — renames, import updates, applying a known pattern to
  several files, boilerplate scaffolding, straightforward
  refactors — dispatch the agent with `model: "haiku"`. Haiku is
  faster and cheaper, and the work doesn't need Opus-level
  reasoning. Reserve the default model for edits that require
  judgment: novel logic, ambiguous specs, cross-cutting changes,
  or anything where getting the design right matters more than
  throughput.
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
use that approach it was explicitly rejected during planning.

---

## Before Each Commit

After each logical unit of work, before committing:

1. **Run linters and type checkers.** Fix any new violations.
2. **Run the tests relevant to the change** (at minimum, the scope
   of the spec's verification command). All must pass — never
   commit on a red state you introduced.

Then commit with a clear message. Keep commits under 400 changed
lines — review effectiveness drops sharply beyond that threshold.
If a logical change exceeds 400 lines, split it into smaller
commits.

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

## Final Review

Before finalizing, launch four review agents **in parallel** — a
single message with four Agent tool calls. Do not self-review —
models fail to correct their own errors.

1. **`staff-reviewer`** (`subagent_type: "staff-reviewer"`) —
   multi-pass review of the code: security, correctness,
   performance, reliability.
2. **`code-quality-reviewer`** (`subagent_type:
   "code-quality-reviewer"`) — rigor signaled vs. rigor enforced:
   contracts not checked at the boundary, stated-purpose patterns
   defeated by usage, defensive code masking contract violations,
   broken encapsulation.
3. **`qa-reviewer`** (`subagent_type: "qa-reviewer"`) — test
   quality, test coverage, and edge case handling.
4. **`compliance-reviewer`** (`subagent_type:
   "compliance-reviewer"`) — did we build what the spec said we'd
   build?

Pass all four the same inputs:
- `diff`: the full feature diff (`git diff <base-branch>...HEAD`,
  where `<base-branch>` is the branch this feature was cut from —
  usually `main` or `master`)
- `spec_path`: the spec file path

When all four return, merge their findings (deduplicate any
overlap), then resolve them:

- **Staff / QA / code-quality findings**: fix all BLOCKING and SHOULD_FIX findings.
  SUGGESTIONS can be deferred or skipped if they're out of scope,
  but note any you skip and why.
- **Compliance deviations** (NON_COMPLIANT): for each — if the spec
  is right and the code drifted, fix the code. If the deviation is
  a reasonable amendment (an edge case the spec missed, a
  constraint that turned out to be wrong), update the spec to
  reflect what was built. Do not finalize while uncorrected
  non-compliance remains.

After fixing, re-run each reviewer whose findings you addressed (or
whose scope your fixes touched) until staff, code-quality, and QA
all return APPROVE and compliance returns COMPLIANT.

---

## Finalize

After verification passes:

1. Run the full test suite (from CLAUDE.md's operational commands)
   to check for regressions beyond the spec's verification scope.
   Fix any regressions before finalizing.

2. Update the spec's "Last updated" date if implementation revealed
   new constraints or edge cases worth recording (or if the
   compliance review prompted spec amendments).

3. Flip the spec's `Status` from `Waiting Implementation` to
   `Implemented` — both in the spec file's header and in its row
   in the Spec Index (`docs/specs/spec.md`). If the spec lives
   under `docs/specs/bugs/`, update the Bugs table; otherwise
   update the Features table.

4. Present a summary:
   - **Branch**: name and commit count
   - **What was done**: 2-3 sentences
   - **Verification**: pass/fail status
   - **Final review findings**: issues caught and fixed by the
     staff, code-quality, and QA reviews
   - **Compliance review**: COMPLIANT, or list of resolved
     deviations
   - **Needs attention** (if any): anything unexpected, deviations
     from the spec, or issues the user should review

**Do NOT push or create a PR.** The user handles this.
