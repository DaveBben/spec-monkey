<!--
SPEC TEMPLATE: the canonical engineering-spec format (the gold standard).
This is the reviewer-facing document AND the spec: ONE artifact, written directly by
create-spec. There is no separate plan. It is outcome-first and works for features,
refactors, renames, and infra alike. Fill the < > placeholders; delete guidance comments
before review.

ALTITUDE RULE: the spec is DESIGN, not implementation.
  In the spec:   what / why, requirements, how it fits, DATA CONTRACTS, edge cases, blast
                 radius, verification, rollout. Everything a human signs off on, and no more.
  In tasks.md:   the per-task file manifest, exact symbols, run commands, ordering, and the
                 implementation APPROACH (which pattern to mirror, the gotchas, within-task
                 ordering). That's the HOW. The spec-decomposer writes it AFTER the human approves.
  Note:          a load-bearing approach DECISION a reviewer must weigh (e.g. "shape X, not Y,
                 because…") is design. It stays here, under Scope decisions. Only the mechanical
                 how-to goes to tasks.md. And there is NO NFR section: non-functional bounds go on
                 the Non-functional line in How it fits, measured ones also get a Success criterion.
  Litmus test:   if a line tells an implementer WHICH file or symbol to edit, it belongs in
                 tasks.md, not here. A reviewer approves direction; they do not approve line edits.

SINGLE SOURCE OF TRUTH: each decision lives in ONE section; everywhere else REFERENCES it by
ID (FR-001, SC-001, a config name), never restates it. Restating is the top cause of bloat
and drift. Two sections saying the same thing → one is a copy; cut it to a reference.

IDs are LIGHTWEIGHT join keys, nothing more:
  - FR-NNN: a functional requirement, one SHALL sentence. No EARS ceremony, no nested
    acceptance-criteria lists. The sentence IS the contract.
  - SC-NNN: a success criterion, one verifiable outcome.
  tasks.md cites these (`implements FR-001 · verifies SC-002`) to trace work back to intent.
-->

---
schema_version: 3                # spec format version; tooling branches on this
id: SPEC-NNN                     # stable handle; tasks/commits reference this
title: <short imperative title>
status: draft                    # draft → reviewed → decomposed → applied → archived
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
owners: [<@handle>]
branch: NNN-<kebab-title>
standards: standards.md          # the constitution (ONE of: standards.md | CLAUDE.md | AGENTS.md)
depends_on: []                   # other SPEC-NNN this needs first
supersedes: []                   # SPEC-NNN this replaces (delta lineage)
---

# Spec: <title>

**Goal (verifiable):** <one sentence stating a delta you can check true or false, e.g. "X moves
from A to B," tied to a metric or an observable outcome that Success Criteria pins down. Not
"improve X". This is the single sentence a reviewer signs off on.>

## 1. The request
<!-- The ask, summarized to a sentence or two, not the full transcript. A reviewer uses it to
     confirm the spec didn't drift from intent. The mechanics live in What I'm building; don't
     restate them here. -->
> <one-to-two-sentence summary of what was asked>

## 2. Why it matters
<!-- The problem, conclusion-first: what's broken or missing, and the cost of leaving it.
     Carries WHO benefits when that's relevant. Keep it short: a few sentences. -->

## 3. What I'm building
<!-- WHAT changes and the rules it must obey: observable behavior, not implementation. -->

**Behavior**: <the runtime flow in a few numbered steps; the shape of the change>

**Requirements**: the observable rules the change must satisfy. Keep the normative **SHALL**, but
write plain English. Name the actor and the action, one condition per sentence, and describe
behavior a reader can picture. Skip implementation or lint trivia, stacked clauses, and noun-form
jargon.
- **FR-001**: The system SHALL <plain, concrete behavior>.
<!-- Good:   The system SHALL mark an article read on the server only after its row is saved.
     Jargon: The system SHALL enforce write-after-persist ordering for read-state idempotency. -->
- **FR-002**: The system SHALL <plain, concrete behavior>.

**Scope decisions to sign off** *(operational or approach calls a human must approve; omit if none)*
- **<decision>**: <the tradeoff, why this way, and the alternative rejected>. *<the question for the reviewer>*

**Out of scope**
- <excluded item>: <why, or where it lives instead>

## 4. How it fits
<!-- Ground the change in the real system so a reviewer can judge fit. -->

**Existing-code facts (verified)**: <numbered facts checked against the actual code, each with
a file reference. These are the load-bearing truths the design rests on, and the reviewer can
spot-check them. Distinguish verified fact from assumption (assumptions go in Verification/rollout).>

**Constraints**: <hard limits that shape the design: platform, volume, must-reuse, compliance.>

**Non-functional**: <the bounds that apply — walk performance, security, reliability, accessibility.
State each with its threshold ("first-run reconcile finishes under N min"; "no PII in logs"); a bound
checked post-ship also gets a Success criterion. This one line replaces a separate NFR section —
write "none" only once you've walked the four categories and none apply.>

**Depends on**: <external services / APIs / other specs / teams the design rides on>

## 5. Data contracts *(omit as "N/A — reason" only if there is no data or type surface)*
<!-- The data shapes and type contracts this change introduces or alters. For a type-bearing
     change these ARE the spec's most precise part: pin them HERE as the source of truth, not in
     a design doc. A typed signature block is the contract; entity prose suffices for simple cases.
     This is design, not implementation: it stays in the spec even though the file manifest doesn't. -->
- **<Entity / contract>:** <what it represents; key fields; relationships>

```python
# typed contract block — the source of truth when the change is type-bearing
class <Name>:
    <field>: <type>
```

## 6. What could go wrong (edge cases)
<!-- The non-obvious failure modes and boundaries, each with a decision: HANDLE / ACCEPT /
     OUT-OF-SCOPE. Cite the FR it relates to. Skip any case an FR already fully pins. Include
     only the branches the requirements don't obviously cover. -->
- **<failure / boundary>**: <what happens> → <decision>. (FR-0xx)

## 7. Who's affected (blast radius)
<!-- Who and what this change touches beyond the code edited. -->
**External**: <users, downstream data consumers, operators / runbooks, other services>
**Internal (code)**: <callers, APIs, shared state that changes; what breaks at build time>
**Unchanged**: <the thing a reader will worry about that this does NOT change; naming it reassures>

## 8. How I know it works
**Success criteria** *(the verifiable definition of done, in plain language, no jargon)*
- **SC-001**: <measurable, technology-agnostic outcome, e.g. "95% of X within 1s", not "uses Redis">

**Verification**: <the test plan, in plain steps a reader can follow. At least ONE integration test
driving the real seam end-to-end is REQUIRED (DB, HTTP, filesystem, queue, or a contract between
two modules), not just units in isolation. Cite the SC/FR each test proves. Exact commands live
per-task in tasks.md, not here. Only a spec with no runtime behavior (pure rename/refactor/docs)
may omit the integration test, with a stated reason.>

**Worked case** *(one concrete end-to-end example, with real values, not "the correct result")*
- **Given:** <real starting state> · **When:** <action> · **Then:** <exact expected values>

**Honest gap** *(state plainly what the tests do NOT prove, and what covers it instead, e.g. a
manual check or production log-watching. Omit only if the tests truly prove everything.)*

## 9. How it ships (rollout & rollback)
**Rollout**: <sequencing; pre-flight checks; the risky moment; what to watch as it goes live>
**Rollback**: <how to undo; what state persists; the bounded cost>

## 10. Tasks
<!-- Pointer only. The per-task breakdown (file manifest, exact symbols, run commands, ordering)
     lives in tasks.md, written by the spec-decomposer AFTER approval. It stays empty until then. -->
→ See [`tasks.md`](tasks.md).

## 11. Activity Log
<!-- Pointer only: the audit trail is git, not a hand-maintained log. -->
→ See git history: `git log -- docs/specs/{slug}/`.

---

# Parse contract (lightweight)

The spec is **human-first Markdown** with a few machine-readable conventions. A parser reads it
without understanding prose.

1. **Frontmatter**: YAML between the leading `---` fences. Holds every queryable scalar
   (`schema_version`, `id`, `status`, `depends_on`, `supersedes`). Never bury one in prose.
2. **Sections**: `^## (\d+)\. <name>`; the canonical set and order are the schema. Cite a section
   by its **header name**, never its number (numbers shift).
3. **IDs are the join keys**: `FR-NNN` and `SC-NNN`, unique within the spec. `tasks.md` cites them
   to build the task → requirement / success-criterion trace. No EARS `#N`, no `F#` manifest ids.
   The file manifest lives in `tasks.md` and owns its own per-task ids there.
4. **Light conventions**: `- **Key:** value` is a key/value field; `**Group**` on its own line
   introduces the list/table/block that follows; a fenced block (Data contracts) is captured with
   its language tag. Everything else is opaque prose keyed by its heading.
