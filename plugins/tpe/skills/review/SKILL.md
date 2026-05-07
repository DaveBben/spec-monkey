---
name: review
argument-hint: "[spec path or base branch]"
description: >
  Spec-grounded code review. Reviews changes against a spec file,
  checking for spec compliance, missed edge cases, constraint
  violations, and logic errors. One general reviewer — not 4
  specialists. Use when you want a second opinion on implemented
  changes before pushing.
disable-model-invocation: true
---

# Review — Spec-Grounded Code Review

Review code changes against the specification that defined them.
Without a spec, AI review is structurally circular (Zietsman 2026).
With a spec, review effectiveness improves by 90.9% (SGCR).

---

## Input

Resolve `$ARGUMENTS`:

1. **Spec path** (`docs/specs/features/{slug}/spec.md`): use as
   the review ground truth. Diff against the branch base.
2. **Slug**: try `docs/specs/features/{slug}/spec.md`
3. **Base branch/commit**: diff against it. Look for a related spec
   in the Spec Index (`docs/specs/spec.md`). If no spec found,
   warn that the review will be ungrounded and less effective.
4. **No input**: diff against main. Search for related spec.

Read the spec (if found) and the diff.

**If no spec is found:** warn the user that the review will be
ungrounded and less effective. Skip spec compliance and edge case
coverage checks — they require a spec to ground against. Focus on
logic/correctness and regression risk only. Mark the verdict as
low-confidence in the output.

---

## Review Process

### What to check (in priority order)

**1. Spec compliance**
- Does the implementation match the spec's Approach?
- Are all Constraints satisfied? Check each one.
- Are all "Do NOT" boundaries respected?
- Were any "Alternatives rejected" approaches accidentally used?

**2. Edge case coverage**
- For each edge case in the spec: is it handled in the code?
- Are there edge cases the spec missed that the code reveals?

**3. Logic and correctness**
- Trace data flow through the changed code
- Check for: off-by-one, null/nil handling, race conditions,
  resource leaks, error handling gaps
- AI-generated code has **2x more error handling gaps** (CodeRabbit)
  — look specifically for missing error paths, uncaught exceptions,
  and silent failures

**4. Regression risk**
- Were files outside the spec's "Files that matter" modified?
- Do the changes affect callers or consumers not listed in the spec?
- Run the verification command — does it still pass?

### What NOT to review

- **Style and formatting** — linters handle this
- **Naming conventions** — linters handle this
- **Architecture decisions** — those were made in `/tpe:spec`
- **Hypothetical performance** — only flag performance issues with
  a concrete trace (e.g., "O(n²) on unbounded collection"), not
  "this could be slow"

### Evidence standards

Every finding must include:
- **Location**: file:line
- **Trace**: the data flow or logic path that leads to the issue
- **Fix**: what specifically to change

Findings without traces get dropped. "This could be a problem"
without showing why is noise, not signal.

---

## Output

```markdown
# Code Review

**Spec**: {spec path or "ungrounded — no spec found"}
**Changes**: {X files, +Y/-Z lines}
**Scope**: {1-2 sentences}

## Issues

### BLOCKING
{Issues that will cause real problems in production}

- `file:line` — [Description]. Trace: [data flow]. Fix: [suggestion]

### SHOULD_FIX
{Issues that should be fixed before merging}

- `file:line` — [Description]. Trace: [logic path]. Fix: [suggestion]

### SUGGESTIONS
{Improvements that are optional}

- `file:line` — [Suggestion with rationale]

## Verdict

**[APPROVE | REQUEST CHANGES]** — [One sentence]
```

Omit empty severity sections. A clean review with 0 findings is a
valid outcome — do not manufacture findings.

**Severity rules:**
- BLOCKING = will cause a production problem
- SHOULD_FIX = should be fixed before merging, but not urgent
- Any BLOCKING or SHOULD_FIX → REQUEST CHANGES
- Only SUGGESTIONS or clean → APPROVE

**False positives erode trust faster than false negatives.** When
in doubt, drop the finding. A clean report developers trust is
worth more than a thorough report they ignore.

---

## Limitations

This review is not a substitute for human review on critical code.

- AI review catches **42-48% of runtime bugs** (Greptile, Martian benchmarks)
- Same-model review has **correlated blind spots** — the reviewing model shares error patterns with the generating model (ICML 2025: 60% error agreement)
- Without a spec, the review is ungrounded and structurally less effective
- For maximum effectiveness, use a different model family for review than was used for implementation
