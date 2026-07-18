# Design brief: a lightweight agentic-coding method (build this as a plugin)

This is a brief for an agent to build a skills plugin. It describes a method for
AI-assisted coding designed to fix the documented failures of spec-driven
development (SDD). Build the skills to enact this method. Working name:
**the method** — rename freely.

---

## Philosophy (the one paragraph)

The code and its tests are the only source of truth for what the system does.
Everything else is either short, disposable steering or dated history. We refuse
to create a long, generated, authoritative prose spec — that single artifact
causes most of SDD's failures. Ceremony scales to the cost of changing a
decision later: nothing for a typo, a real (but brief) scoping pass for something
expensive to reverse. The human authors intent and ratifies the tests; the agent
compiles that intent into the failing test contract but never freezes it unratified,
so the source of truth is never the agent's unilateral invention. Verify by driving
the real feature *and* by an adversary reading
the code — never by a self-graded green check. Keep only the code, the tests, and
the *why*; throw away everything else at merge.

**The core move:** a spec fuses three jobs that want opposite things — steering
(short, disposable), verification (executable, deterministic), and rationale (the
"why", durable). Split them into three artifacts, each with the right lifespan.
Verification takes its strongest form: a frozen test contract authored before the code
and binding on the build, not a prose plan nobody re-checks.

---

## Actors

- **Human** — authors intent and drivers, dispositions edge cases, ratifies the
  decision, reads diffs. The only author of the source of intent.
- **Builder** — the implementing agent. Proposes approach and edge cases; writes
  tests and code. Never the final grader of its own work.
- **Reviewer** — a *separate* agent in a fresh context. The adversary. Attacks the
  diff for correctness, security, and fake tests.

---

## Artifacts and their lifespans

| Artifact | Home | Lifespan | Holds |
|---|---|---|---|
| **Intent** | Plan file at `<plan_dir>/<slug>/plan.md` (gitignored), from the `plan_template` | Disposable — dies at merge | Goal, a pointer to the contract tests, non-test verification, out-of-scope. Short: the requirements live in the tests, not restated here |
| **Slice checklist** | the `task_list` path (default `.tests-as-specs/task-list.md`, gitignored) | Ephemeral — deleted at merge | The thin-slice to-do list for *this* change; session-resume memory |
| **Decision log** | One file per decision under `adr_dir` (default `.tests-as-specs/adrs/`), ADR-style | Durable — kept forever | The *why*: rationale, rejected alternatives, tradeoffs, and consciously-unhandled edge cases |
| **Contract tests + code** | The repo | Durable — the source of truth | What the system does. The contract is authored in scope as failing stubs (each pinning an outcome), wired by the build, and is the executable form of every requirement |

Hard rules:
- **Constraint values live in tests, never as prose in the log.** The log records
  *why* a constraint exists, not its value — else you have two sources of truth
  that drift (the rotting second-spec).
- **Never use persistent/global agent memory for slice state.** Slice memory is
  branch-local and dies at merge. The durable slot belongs to tests and the log.
- Test for where something goes: *would a developer six weeks from now want to
  read this?* Yes → log or test. No → intent or slice checklist (disposable).

---

## The flow, step by step

### Step 0 — Triage (always, cheap)
On any change request, decide the lane by **cost of change**:
- **Just build it** — trivial, one sane approach, no blast radius (typo, null
  check, copy tweak, version bump). Skip ceremony. Go straight to Build (Step 4).
- **Ceremony** — expensive to reverse: crosses seams, touches a data
  shape/migration, changes an external contract, crosses a trust boundary, or
  needs sign-off. Run the full flow.

State the lane and why in one line. Do not manufacture ceremony to look careful.

Look before you price it: you can't judge cost-of-change without reading what the
change touches, so a quick blast-radius look precedes the lane call, and anything you
can't bound at a glance runs Probe (Step 1) first.

When you can't cleanly place the lane, choose Ceremony: the costs are asymmetric,
and guessing "just build it" on a change that crosses a trust boundary skips the
whole safety apparatus. Any touch of a trust boundary, authorization, a data
shape/migration, or an external contract is the Ceremony lane at any size. Triage is
provisional, not a commitment: you often can't know the lane before you understand
the change (problem 6), so if Probe or Build reveals blast radius you didn't see,
re-triage.

### Step 1 — Probe (only if the change is unfamiliar)
If the builder doesn't understand the current-state code well enough to scope
correctly, build a throwaway spike to *learn*, report what was learned, then
**delete the spike**. You cannot write correct acceptance criteria for something
you don't understand yet. Skip this step when the ground is already known.

### Step 2 — Scope ceremony (human authors; agent interviews)
Before interviewing, the agent reads the decision log for prior decisions in this
area, so the interview doesn't re-open a settled call or contradict a chosen
constraint. The skill interviews the human — hardest-uncertainty first — and the
**human answers**. The agent does not invent requirements. The agent's active jobs here:
1. Ask the questions that pin the **goal as a checkable delta** and the
   **constraints that actually bind**.
2. **Surface edge cases** via risk lenses (failure & scale, trust boundary,
   malformed/hostile input, concurrency, implied work). This is where the agent
   adds real value — the human dispositions each surfaced case:
   - **Handle** → becomes an acceptance criterion → becomes a test.
   - **Accept** (known gap) or **Out-of-scope** → a receipt in the decision log.
3. **Author the failing tests now:** the executable form of the requirements,
   written before the code, frozen as the contract the build must satisfy. Author it
   as failing stubs: each a named test whose description pins the concrete expected
   outcome, with a body that asserts false. The build wires each stub into a real
   assertion that checks exactly that outcome and never changes it. The test-author (scope, with the
   human) is deliberately not the test-passer
   (build); an implementer that writes its own tests writes tests it can pass. Not
   every requirement reduces to a unit test: a performance NFR becomes a benchmark or
   load test, a "no unauthorized path" property routes to the reviewer's security read,
   and genuinely exploratory surface (UI feel) becomes a written manual observation
   script a human runs. The rule is only that every requirement gets an *executed*
   check, never prose nobody runs. Derive expected values independently, never a
   snapshot of what the code will emit.

Output: the **failing test contract** (stubs pinning outcomes), plus a short **intent**
in a local plan file: goal, a pointer to those tests, any non-test verification,
out-of-scope. The intent stays short because the tests carry the requirements; do not
restate them as prose.

### Step 3 — Log the why, then pause for review
Write a receipt to the **decision log**: the rationale, the rejected alternatives, and
the tradeoffs the decision imposes. A consciously-unhandled edge case is its own entry.
You read this log before interviewing (Step 2), so you know what was decided before.
The log is one file per decision under `adr_dir` (default `.tests-as-specs/adrs/`),
which keeps it rebase-friendly: two branches adding decisions never touch the same file.

Then pause. The plan and the failing contract are ready, so tell the human where the plan
is saved and let them choose. If they want the team to review the direction, they open a
PR now and launch the build later with that plan path. If they don't, they continue in the
same session and the build starts. The review gate stays optional and human-driven, not a
policy the tool enforces.

### Step 4 — Build in thin slices
Work one vertical slice at a time (the smallest change that produces observable
behavior). The contract tests already exist and are failing; the builder does not
author them. For each slice:
1. Builder maps the slice to the contract tests it must turn green. The contract's
   expected outcomes are frozen: the builder wires each stub into a real assertion that
   checks exactly the outcome its description pins, makes real tests pass, and never
   weakens, skips, or deletes one. A contract test that is wrong goes back to scope,
   not patched to pass.
2. Builder implements until those tests are green, adding finer tests below the
   contract as needed.
3. Prove the slice before starting the next. Track slices in the `task_list`
   file (default `.tests-as-specs/task-list.md`, gitignored; dies at merge).

Bug-fix protocol (explicit): reproduce with one failing test, fix, keep the test.
No story fan-out. A one-line bug is a one-line fix plus a regression test.

### Step 5 — The two-part gate (both required; they guard each other)
A green check alone is never trusted. The two gates run at **different times**,
because they cost different amounts and catch different things.

- **Gate A — behavioral. Runs after *every* slice.** Run the real feature end to
  end with real inputs and *observe* the outcome — including an abuse case. Not
  "tests green"; demonstrated behavior. This isn't optional per-slice: proving each
  slice before the next *is* Gate A, and it's cheap (you just run it). It stops you
  building slice 3 on a slice 1 that silently doesn't work.
- **Gate B — adversarial review (fresh-context Reviewer). Runs once, on the
  complete diff, before merge.** Reads the actual diff hunting for where it's
  wrong, unsafe, or mismatched to intent. Standing mandate on every run:
  - **Correctness** against the intent.
  - **Security**: trust boundaries and authorization on the real code.
  - **Test integrity**: every test would actually fail if the code were broken —
    no assertions on constants, no self-verifying mocks, no stubbed-out real path.

  It runs at the end, not per slice, because its best catches — security holes,
  cross-slice inconsistency — are whole-change properties invisible in one slice,
  and a fresh-context pass is expensive to repeat.

  **Escalation — review a foundational slice early.** If a slice lays a shared
  abstraction, opens a trust boundary, or changes a data shape that later slices
  build on, run Gate B on *that* slice before building on it. Cost-of-change
  applied to review timing: catch a bad foundation before three floors sit on it.

This self-scales: a 1–2 slice change collapses the per-slice proof and the final
review into effectively one pass; a 4–5 slice change with a risky foundation earns
one early Gate B plus the final one — exactly where the extra pass pays off.

**Gate B has a floor, and it scales with cost of change.** A fresh-context Gate B is
mandatory whenever the change was scoped, or touches a trust boundary, authorization,
a data shape/migration, or an external contract, at any size. A trivial change on the
just-build-it lane with none of that surface does not get a separate fresh-context
pass: there, Gate A plus the human reading the diff is Gate B. This rules out both
failure modes at once. A typo doesn't earn a full adversarial review, and the
high-volume lane still gets one independent read.

**Gate B's independence is contextual, not architectural.** A fresh-context Reviewer of
the same model shares the Builder's blind spots. It catches self-justification, context
poisoning, and tests that don't bite; it does not catch a bug class the model is blind
to everywhere. A human or a different model closes that gap. Without one, Gate B is
strong code review, which is the floor, not a failure.

Gate A catches broken code; Gate B catches hollow tests and untested boundaries.
Ship only one and you keep the false sense of rigor while removing what made it
real.

### Step 6 — Route findings, then merge and clean up
- Reviewer FAIL → route each finding: "fix code" back to Builder; "the ask is
  wrong" back to the human/ceremony. Re-run Gate A on the affected slice and the
  final Gate B on the updated diff.
- Gate B green (and every slice's Gate A green) → the human merges (never the
  agent on its own read).
- **At merge, keep code + tests + the decision-log receipt. Delete the intent,
  the probe spike, and the `task_list` slice checklist.** Nothing that can rot
  survives.

---

## Task breakdown across developers (two levels)

Do **not** build a task-tracking system — that reinvents Jira, which is the
disease. Use two levels, only one of them shared:

- **Level 1 — across changes (team-visible, durable): the existing ticket
  tracker.** One expensive change = one ticket = one trip through this flow. A
  large initiative is decomposed *at ceremony time* into a **few coarse tickets**
  with dependencies noted (ticket B blocks on ticket A) — never a fine-grained
  2k-line task tree. Each ticket is independently flowable by a different
  developer. The decision log records *why* the seams were cut there.
- **Level 2 — within one change (private, ephemeral): the `task_list` file** (default `.tests-as-specs/task-list.md`, gitignored). The
  slices of a single ticket are one developer's working memory. Not shared team-
  wide; dies at merge.

**The rule that keeps coordination in the tracker:** one shippable slice = one
ticket = one branch. If two slices need two developers in parallel, they are two
tickets, not two slices — split them so the coordination surfaces in the tracker.

**Shared durable context across the team** is the committed decision log (read via
git). The tracker holds what's in flight; the log holds why past calls were made;
the code and tests hold what's true now.

---

## The decision log (spec)

One file per decision under `adr_dir` (default `.tests-as-specs/adrs/`), committed and
durable. Read it before scoping a change in the same area, so you don't re-open a
settled decision or contradict a chosen constraint.

- One file per decision, named `<date>-<slug>.md`. This is rebase-friendly: concurrent
  branches never touch the same file. Don't rewrite a past decision; add a new one that
  supersedes it.
- Log sparingly: one file per genuinely expensive or irreversible decision, or per
  consciously-unhandled edge case. The bar is high; most changes add none. If it starts
  reading like a spec, you are logging too much.
- Per file: date · the decision · why · rejected alternatives · the tradeoffs the
  decision imposes.
- It records *why*, never *what is true now*. To learn current behavior, read the code
  or run the tests.

Example (`2026-07-17-password-reset-tokens.md`):
```
# 2026-07-17 · Password reset tokens: single-use, 1h TTL.

Why:       Replay defense: a leaked link must not work twice.
Rejected:  JWT-in-URL (can't revoke); multi-use links (replay risk).
Tradeoffs: A user who opens the email after an hour must request a new
           link; we accept that friction for replay safety.
```

---

## Skills to build (suggested shape — the builder decides final structure)

At minimum:
1. **`scoping`** — explore the blast radius (and past ADRs), then triage by
   cost-of-change; on the ceremony lane, interview the human, surface + disposition edge
   cases, author the failing-stub contract, emit the intent to the plan file
   (`<plan_dir>/<slug>/plan.md`) and a receipt to the decision log, then pause and let the
   human decide whether the team reviews the direction before code. Trigger: an
   expensive-to-change request.
   Explicitly *does not* fire on trivial changes.
2. **`building-from-plans`** — thin-slice implementation on an isolated branch that makes the frozen
   contract pass (wiring each stub into a real assertion, never weakening its outcome);
   `task_list` slice checklist (gitignored, config-driven); runs **Gate A behavioral after every slice**; defined
   bug-fix protocol; hands the merge to the human.
3. **`verifying-work`** — the fresh-context Reviewer running **Gate B once on the complete
   diff** (and on a foundational slice early, when flagged): adversarial diff review
   with the standing correctness + security + test-integrity mandate, including that the
   frozen contract is intact and each stub was wired faithfully. Report-only,
   PASS/FAIL, evidence on both sides (`file:line` + the contract test or intent line).

Plus a **decision-log convention** (one ADR file per decision) that
`scoping` writes to and `building-from-plans`/`verifying-work` read.

Keep skill bodies lean and harness-neutral; push detail into reference files.

---

## Honest costs and non-goals (state these; don't hide them)

- **Gate B costs a second agent pass** on every expensive change. Real cost, worth
  it — it's where correctness actually comes from.
- **The decision log needs discipline.** One file per real decision, and no more.
  Undisciplined, it bloats back into a spec.
- **The human must author intent and read diffs.** This is deliberate — it's the
  only reliable defense against fabricated requirements. If you won't read the
  code, no method saves you.
- **If intent is wrong, you get a correct build of the wrong thing.** No method
  fixes that. This makes "it works" *demonstrated* and "conformance" *executable*;
  it does not make your intent correct.
- **Not a claim of proven results.** The design reverts to evidenced practices
  (vertical slices, code review, tests, ADRs) plus thin AI glue, and is
  lightweight enough to A/B against alternatives. Measure it; don't trust the
  feeling of progress.
- **"Tests are the source of truth" is 90% true, not 100%.** Some requirements resist
  a green unit test: performance NFRs need a benchmark, "no unauthorized path" is a
  universal negative you can't assert, and UI feel stays a human's judgment. Those
  route to a heavier executable check, the reviewer's read, or a manual script. The
  claim is "every requirement gets an executed check," not "every requirement becomes
  a passing unit test."
- **The contract only binds if the builder can't change what it asserts.** The builder
  wires each stub into a real assertion, but a builder free to weaken the pinned outcome
  has a suggestion, not a contract; the easiest path to green becomes moving the
  goalposts. The freeze (build never weakens a stub's expected outcome) plus the
  reviewer's contract-integrity check are what make the tests-as-spec move real rather
  than faked-done with extra steps.
