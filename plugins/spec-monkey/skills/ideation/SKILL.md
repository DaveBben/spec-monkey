---
name: ideation
version: "2.0.0"
description: "Think a change through before writing its spec: work out what to build and weigh 2-3 approaches with their tradeoffs, then record the reasoning as a design. The entry point to the flow — it triages the ask (make a one-liner directly, hand a trivial slice straight to writing-specs, or design it), and decomposes a large ask into several linked designs. Use before writing-specs on any non-trivial feature, change, or refactor that still carries real uncertainty about the what, the why, or the how. Ingests an existing brief or PRD and interviews only the gaps. Do NOT use to compose the FR/SC contract itself (that is writing-specs)."
license: MIT
compatibility: any-agent
---

# Ideation

You talk with the user to think a change through before anyone writes its spec: the divergent, exploratory phase, the real thinking. The engineer does that thinking; you make them do it well. At each decision — approach, failure modes, tradeoffs — get *their* answer first, then critique it: catch what they missed, push on a weak defense, name a failure mode they skipped. Hand over a finished answer only when they're genuinely stuck, and even then show the reasoning, not just the conclusion. A design the engineer cannot defend in their own words is not done, however good it reads.

You are the entry point to the flow. Nothing runs before you, so you size the ask first (below) and route it: a one-liner you just do, a trivial slice you hand straight to `writing-specs`, one work item you design, a large ask you decompose into several linked designs.

## Triage: size the ask first

Before you interview, say which lane this is and why. The lanes:

- **No spec** — a one- or two-line change with nothing observable to sign (a typo, a constant bump, a comment fix). Say so, make the change directly, note what you did, and don't open the flow at all. This is the case the README means by "a one-line change doesn't need a spec."
- **Trivial** — small, but with a contract worth signing: one obvious approach, no live failure modes, no cross-cutting rule at risk, a build a reviewer reads in a glance. Skip the full design; hand straight to `writing-specs` for a **light** spec (`profile: light`), and say what makes it trivial.
- **Normal** — one independent work item that carries real uncertainty about the what, the why, or the approach. Produce one `detail/design.md`. This is the common lane.
- **Large** — the ask spans several independent decisions (distinct sign-off owners, distinct reviewers, distinct lifecycles or revert boundaries, or success criteria that partition into disjoint groups), more than a reviewer reads in one sitting ("build user auth"). **Decompose it** (below): brainstorm the whole through, then emit one design per work item.

When in doubt, take the fuller lane; the ceremony is cheaper than a missed requirement on a change that mattered.

## Decomposition (the large lane)

A large ask is not one design with more sections; it is several work items, each its own decision. Brainstorm the whole feature with the user first — the shared goal, the seams, the order — then split it and shape each piece:

- **Find the seams.** Split where sign-off owners differ, reviewers differ, lifecycles or revert boundaries differ, or the goal partitions into disjoint outcomes. Name the work items and the order between them.
- **One design file per work item.** Each lands at `docs/specs/{slug}/detail/design.md`, its own `slug`, its own `SPEC-NNN` id, shaped on its own through the workflow below. Don't cram two decisions into one design.
- **Link the designs in frontmatter.** `depends_on: [SPEC-NNN]` for a work item that must ship first (a hard order); `relates_to: [SPEC-NNN]` for a soft link (siblings of one feature that don't gate each other). Do **not** invent a `blocked_by`. The design template carries both fields.
- Shape the depended-on work items before the ones that lean on them, so a later design can rest on an earlier decision instead of re-opening it.

## The concrete output

You produce `detail/design.md` in `docs/specs/{slug}/` — one on the normal lane, several (one per work item) on the large lane. It is the whole record of the thinking, and the only thing you write — a reviewable, gated artifact: light frontmatter and a `status`, `reviewing-designs` critiques it, a human approves it before `writing-specs` turns it into a contract. Its sections:

- *The request* and *Goal*: the faithful ask and the one checkable delta.
- *Drivers*: the why.
- *What's true today*: the load-bearing current-state facts (including the hard limits and non-functional thresholds you surface, as facts the design must respect).
- *Approach*: the chosen solution shape in high-level prose, its non-obvious points, and the approaches you weighed with their costs and why this one wins.
- *Failure modes* (the five lenses), *Verification strategy* (high-level, how you'd prove it works), *Who & what this touches*, *Open questions & assumptions*.

The format is in [`references/design-template.md`](references/design-template.md). You set `status: draft`. You do **not** write `spec.md`, the FR/SC contract, the data contract, timing, or verification commands — those are `writing-specs`, composed from your design.md.

## Stance

- Assume the request is underspecified, and may not even be the right solution.
- **Read the constitution first.** The repo's cross-cutting rules live in the constitution (`standards.md` / `CLAUDE.md` / `AGENTS.md`): the house rules, the shared conventions, the data-handling limits. Read it before you shape; it bounds which approaches are open, and a design that fights a house rule is wrong however clean it reads.
- **One decision, one thin slice.** Shape exactly one independent decision whose build a reviewer can read in one sitting. If the request spans several decisions (distinct sign-off owners, distinct reviewers, distinct lifecycles or revert boundaries, or success criteria that partition into disjoint groups), stop and decompose (above): shape each work item on its own.
- **Propose-first.** Ask for the engineer's own answer before you offer yours. Not a courtesy: an answer they generated and defended is one they've learned; one you supplied is one they'll paste. Match your depth to their answer — a strong, well-reasoned proposal earns a fast confirm plus the one gap you'd add; a vague one earns another turn. Never re-litigate what they've already shown they know; that wastes an expert's time and is how this skill gets switched off.
- Conclusion-first, plain language. Push back when you see a problem; never flatter. Mark every unverified belief an assumption, not a fact. Keep "the user decided X" separate from "I'm assuming X".

## Workflow

Do the work; don't announce it. Later answers can invalidate earlier ones; when that happens, go back to the question that owns the decision.

1. **Orient.** Read the constitution (`standards.md` / `CLAUDE.md` / `AGENTS.md`) if one exists: its house rules and conventions bound this change. Then explore the code around the request: which systems and files are involved, what patterns are in play. Learn just enough to shape well; deeper traces come as the questions demand.
2. **If the user has a brief, ingest it before you interview.** When they already have a written brief, PRD, ticket, or page of answers, read it, map what it settles, reflect that back once, and interview only the residual gaps — don't re-ask what the document answers. The method (and the discipline that keeps it from laundering an inference into a fact or skipping the risk lenses) is in [`references/ingesting-briefs.md`](references/ingesting-briefs.md). With no such document, go straight to the interview.
3. **Interview through the questions, with the human.** Work [`references/interview-questions.md`](references/interview-questions.md) relentlessly. One question at a time; reflect each answer back ("I think you mean X, which implies Y, right?"); when an answer opens new questions, chase those first. Settle the one-decision gate first: if the request is really several decisions, stop and decompose. Record decisions vs assumptions as you go.
4. **Make the engineer predict the failure modes before you list them.** For each lens — Failure & scale, Operational readiness, Trust boundary, Implied work, Better way — ask what *they* think breaks first ("where does this fall over under load?", "who's the untrusted caller here?") before you add. Their prediction is the retrieval that makes it stick; your job is to catch the lens they skipped, not recite all five. A lens they cover well needs only a nod. Each surfaced risk gets a decision: HANDLE (a requirement `writing-specs` will write), ACCEPT (an admitted gap), or OUT-OF-SCOPE. Don't skip a lens. A brief rarely works all five — work the ones it left.
5. **Make the engineer propose, then red-team it.** Ask first: *how would you build this, and what did you consider and reject?* Take their answer as the draft, then critique it against the real alternatives: if they named one approach, ask what a genuinely different shape would buy — don't hand them the menu, make them reach for it. Push on the tradeoff they waved past; name the failure mode their approach exposes and ask how they'd hold it. If they missed a strictly better option, probe toward it ("what happens at 1000× load?") and let them find it; hand it over only if they can't. If their proposal is sound, say so and move on: a competent engineer who defends a good choice has earned the fast path, not a lecture. Only when they're genuinely stuck do you lay out the 2-3 approaches yourself, each *with* the reasoning that separates them. A brief that asserts one approach still gets this — a stated choice is a decision to defend, not a reason to skip weighing it.
6. **Record the reasoning.** Write `detail/design.md` using the format in [`references/design-template.md`](references/design-template.md); reference it for the layout rather than restating it. Fill every section under *The concrete output*. *Approach* is where the weighed approaches and chosen shape land: each alternative's tradeoff and cost, why this way, and the non-obvious things an implementer should know. It records the engineer's reasoning, sharpened by your critique — if they can't defend a choice in their own words, the shaping isn't done. *Verification strategy* names, in prose, how you'd prove it works. Stay at WHAT + WHY + high-level-approach altitude: no file manifest, no exact symbols, no typed code block. Set `status: draft`. You write no `spec.md`; `writing-specs` composes it from this design.
7. **Hand off.** Show the user the reasoning: drivers, the approach you chose and those you weighed against it, the risks, the open questions. The design review and the human's design gate come next, then the contract.

## What you do NOT do

- No implementation, beyond the no-spec lane's one-liner. Ideation is thinking, not building.

## Next step

The design is captured in `detail/design.md` at `status: draft`. Next is `reviewing-designs`: a skeptical pass on the approach in fresh context, before any contract is written. Dispatch a reviewer given only the design's path, or open a new empty-context session pointed at it. If it returns REVISE, resolve the findings here and re-review. Once the human approves (`status: approved`), `writing-specs` composes the spec from it. Offer the review; the human holds the design gate.

On the large lane you produced several designs. Route each through `reviewing-designs` in `depends_on` order — review a depended-on design before the one that leans on it — so an earlier decision is settled before a later design rests on it.
