---
schema_version: 3                # spec format version
id: SPEC-NNN                      # a stable id, e.g. SPEC-014
title: <short imperative title>
status: draft                    # leave as draft until it's reviewed
created: <YYYY-MM-DD>
standards: standards.md          # the repo constitution: standards.md | CLAUDE.md | AGENTS.md
---

# Spec: <title>

<!-- This spec is DESIGN, not implementation. The file manifest, exact symbols, and run commands
     do NOT go here. The spec-decomposer writes them into a sibling tasks.md after approval.
     If a line names which file to edit, it's in the wrong file. Data contracts DO stay (types
     are contracts a reviewer signs). Full guidance: the handling-specs skill. -->

**Goal (verifiable):** <one sentence stating a delta you can check true/false — "X moves from A to
B", tied to an observable outcome. Not "improve X".>

## 1. The request
> <what you're building or changing, summarized in a sentence or two>

## 2. Why it matters
<!-- The problem, conclusion-first: what's broken or missing, and the cost of leaving it. -->

## 3. What I'm building
**Behavior** — <the runtime flow in a few steps>

**Requirements**
- **FR-001** — The system SHALL <required behavior>.

**Out of scope**
- <item> — <why it's excluded>

## 4. How it fits
**Existing-code facts (verified)** — <facts checked against the real code, with file refs>
**Constraints** — <hard limits: platform, volume, must-reuse, compliance>
**Non-functional** — <performance / security / reliability / accessibility, each a bound with its threshold; "none" if none apply>
**Depends on** — <external services / APIs / other specs>

## 5. Data contracts *(omit as "N/A — reason" if no data or type surface)*
```python
# typed contract block — source of truth when the change is type-bearing
```

## 6. What could go wrong (edge cases)
<!-- Non-obvious failures/boundaries, each with a decision: HANDLE / ACCEPT / OUT-OF-SCOPE.
     Cite the FR it relates to; skip cases an FR already pins. -->
- <failure / boundary> → <decision>. (FR-0xx)

## 7. Who's affected (blast radius)
**External** — <users, downstream consumers, operators / runbooks>
**Internal (code)** — <callers, APIs, shared state; what breaks at build time>
**Unchanged** — <the thing a reader will worry about that this does NOT change>

## 8. How I know it works
**Success criteria**
- **SC-001** — <measurable, technology-agnostic outcome>

**Verification** — <the test plan; at least one required integration test through the real seam.
Cite the SC/FR each proves. Exact commands live per-task in tasks.md.>

**Worked case**
- **Given:** <real starting state> · **When:** <action> · **Then:** <exact expected values>

**Honest gap** — <what the tests do NOT prove, and what covers it instead>

## 9. How it ships (rollout & rollback)
**Rollout** — <sequencing; pre-flight checks; the risky moment; what to watch>
**Rollback** — <how to undo; what persists; the cost>

## 10. Tasks
→ See [`tasks.md`](tasks.md).  <!-- the spec-decomposer writes it after approval -->

## 11. Activity Log
→ See git history: `git log -- docs/specs/{slug}/`.
