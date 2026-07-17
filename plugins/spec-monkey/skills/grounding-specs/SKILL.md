---
name: grounding-specs
version: "1.9.0"
description: "Establish the project spec: the one architecture document every work-item spec grounds on. Runs an interview to capture the shared data contracts, system-wide invariants, trust boundaries, and hard architectural constraints, plus the planned work items and their order. Use once at a project's start before any work-item spec, or when an existing codebase has no project spec yet. Produces a living, versioned document the human approves. Do NOT use for a single feature, change, or work item — shape and write that with shaping-specs then writing-specs."
license: MIT
compatibility: any-agent
---

# Grounding Specs

You talk with the user to establish the **project spec**: the shared architecture every work-item spec grounds on. There is **one artifact**: `docs/specs/project/spec.md`.

It holds what every work item shares and must obey: the canonical data contracts, system-wide invariants (`INV-NNN`), trust boundaries, and hard architectural constraints. It does **not** hold work-item requirements — those live in each work-item spec (`writing-specs`), which links up by `parent` and cites these `INV-NNN` rather than restating them.

## The key idea: shared ground, not feature detail

You draw the ground the features stand on, not the features. Capture only what is genuinely shared and what you're confident about now. The document is **living**: thin at first, amended as the work reveals its real shape. Front-loading a decision you can't yet make is how a project spec becomes waterfall.

## Stance

- **One home for shared decisions.** A fact every work item needs lives here once; work-item specs reference it by ID, never copy it.
- **Thin and honest.** An invariant you cannot defend today is an assumption, not an invariant. Say so.
- **No work-item requirements.** No `FR`/`SC` here. A requirement that belongs to one work item has leaked up; push it down into that item's spec.
- **Architecture, not house rules.** Hard tech and platform calls belong here because they *are* the architecture. Coding conventions (lint, formatting, test style) belong in the constitution (`standards.md` / `CLAUDE.md` / `AGENTS.md`), not here.
- Conclusion-first, plain language. Push back when you see a problem; never flatter. Mark every unverified belief an assumption, not a fact.

## Workflow

Do the work; don't announce it.

1. **Orient.** Read the constitution (`standards.md` / `CLAUDE.md` / `AGENTS.md`) and scan the codebase. On an **existing** codebase, shared contracts and invariants are *discovered, not invented*: read them out of the code, then ratify each with the human (see *Reading invariants out of existing code*). On **greenfield**, elicit them in the interview.
2. **Interview through the questions.** Work [`references/interview-questions.md`](references/interview-questions.md) relentlessly. One question at a time; reflect each answer back ("I think you mean X, which implies Y, right?"); when an answer opens a sub-decision, chase it before moving on. Record decisions apart from assumptions.
3. **Compose the project spec** at `docs/specs/project/spec.md` from [`references/project-template.md`](references/project-template.md). Give every shared fact an ID (`INV-001`, a named entity). Stay at WHAT + WHEN altitude; the exception is the architectural constraints, where a concrete platform or tech call is the point. Set `status: draft`, `version: 1`.
4. **Map the work.** List the planned work items and their `depends_on` order under *Work items & sequencing*. This is the cut plan, not a promise; it changes as the work teaches you the real shape.
5. **The one human gate.** Show the user the spec, its invariants, constraints, and residual risks. Suggest a fresh-context review first (`reviewing-design` handles a project spec): dispatch a reviewer given only this spec's path, or open a new session pointed at it. Approval is the human's alone, though you may record it. On an explicit, unsolicited go-ahead, restate the invariants and constraints they're signing off, then record the gate: `status: approved`, `approved_by` their handle, `approved_date` today. Never approve on your own judgment, never read approval into silence, never nudge them toward it.

## Reading invariants out of existing code (brownfield)

On an existing codebase you do archaeology, not elicitation: the shared rules are already enforced in the code; surface and ratify them, don't invent them. Bound the sweep and don't ratify accidents.

- **Where the invariants hide.** The schema and migrations (uniqueness, foreign keys, not-null, enum sets); validation at the trust boundary (which inputs are rejected); auth and permission checks (who may do what); and the tests that assert a rule holds ("must never…", "always…"). These four are where a system-wide rule is actually enforced.
- **Bound the sweep to the shared entities.** Ground the entities that more than one work item touches — the ones headed for *Shared data contracts* — not every type in the repo. A rule about a purely local type is not a project invariant.
- **A property the code happens to satisfy is not yet an invariant.** What holds today may be an accident, not a guarantee. Present each as a *candidate* — "the code never writes two splits for one example; should that always hold?" — and let the human ratify or wave it off. Don't promote an observed coincidence to `INV-NNN` on your own read.

## Amending an approved project spec

Once approved, a shared change (a new invariant, a new entity, a contract change) is an **amendment**, the highest-blast-radius edit in the system. It fans out to every work item.

- Make the change here, bump `version`, and re-approve.
- Flag every work-item spec written against the old version to re-check against the new one.

`shaping-specs` routes a work item that needs a shared change back here before it builds. Do not let a work item invent a shared fact locally; that is how the ground drifts.

## What you do NOT do

- No implementation, and no work-item requirements: a work item's `FR`/`SC` live in `writing-specs`.

## Next step

`shaping-specs`: think each planned work item through, then `writing-specs` composes its spec on this ground. Offer it once the human sets `status: approved` — the ground must be laid before a work item can stand on it. Point them at the first item in *Work items & sequencing*.
