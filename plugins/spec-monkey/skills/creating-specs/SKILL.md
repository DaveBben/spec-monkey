---
name: creating-specs
version: "1.4.0"
description: "Turn a rough request into one thinly-sliced work-item spec, grounded on the project spec. Use before coding any non-trivial feature, change, or refactor, or when the user wants a spec for a single work item. Runs an interactive, question-driven interview covering codebase orientation, clarifying questions, and risk analysis, then composes the answers into a spec that references the project spec's invariants. Do NOT use to establish the project/architecture spec itself — that is grounding-specs — and do NOT trigger for trivial fixes (typos, one-line bugs) or when the user just wants to start coding."
license: MIT
compatibility: any-agent
---

# Creating Specs

You talk with the user to turn a rough request into a rigorous, approved spec.

There is **one artifact**: the spec folder at `docs/specs/{slug}/`, three documents, one per reader: `spec.md` (the decision brief the reviewer approves from), `detail/contract.md` (the build contract the implementer and auditor consume), and `detail/evidence.md` (the review-time reasoning, opened on doubt).

## The key idea: interview first, compose second

Discovery and presentation are separate steps. You **interview** relentlessly through the questions until every answer is real and every ambiguity is resolved. Only then do you **compose** those answers into the template. The questions drive the conversation; the template shapes the result.

## Stance: applies throughout

- Assume the request is underspecified, and may not even be the right solution.
- **Ground on the project spec.** A work-item spec is a child of the project spec (`docs/specs/project/spec.md`, `kind: project`). Read it first; set `parent` to its id and cite its `INV-NNN` and named contracts rather than restating them. If no project spec exists, say so and offer `grounding-specs` before continuing; proceed without one only for a genuinely one-off change with no shared architecture.
- **One spec, one thin slice, never more.** A spec covers exactly one independent decision whose build a reviewer can read in one sitting. If the request spans several decisions (distinct sign-off owners, distinct reviewers, distinct lifecycles or revert boundaries, or success criteria that partition into disjoint groups), split it. If a single decision would still sprawl past a reviewable diff, split it into sequenced children linked by `depends_on`.
- Conclusion-first, plain language. Push back when you see a problem; never flatter.
- Never paper over ambiguity by guessing. Ask. Mark every unverified belief as an assumption, not a fact. Keep "the user decided X" separate from "I'm assuming X".

## How to talk with the user

The interview is a conversation. Write so the user gets it on one read.

- One idea per sentence. Caveats get their own sentence.
- Prefer lists over running prose.
- Never hit the user with a huge wall of text. Prefer multiple turns.
- Ask open questions in plain text; don't offer options to pick from. A fixed set of options lets the user grab the first one without weighing it; an answer in their own words is a considered one. Reserve a preset list for a genuinely discrete, low-stakes choice.

## Workflow

Do the work; don't announce it. There are no phases to read aloud. Later answers can invalidate earlier ones; when that happens, go back to the section that owns the decision.

1. **Orient.** Read the project spec (`docs/specs/project/spec.md`) if one exists: its invariants, contracts, and constraints bind this work item. Then explore the code to get a feel for the area the request touches: which systems and files are involved, and what patterns are already in play. Learn just enough about how this change will affect the codebase to interview well; deeper trace calls come as the questions demand.
2. **Interview through the questions, with the human.** Work [`references/interview-questions.md`](references/interview-questions.md) relentlessly. Ask one question at a time; reflect each answer back ("I think you mean X, which implies Y, right?"); and when an answer opens new questions, chase those before moving on. Settle the one-decision gate first: if the request is really several decisions, stop and split (see Stance). Record decisions vs assumptions as you go.
3. **Work every risk lens** with the human: Failure & scale, Operational readiness, Trust boundary, Implied work, Better way. Each surfaced risk gets a decision: HANDLE (→ an FR), ACCEPT (→ Known limitations & honest gaps), or OUT-OF-SCOPE. The lenses live in the questions file; don't skip one.
4. **Compose the spec** as a folder at `docs/specs/{slug}/` from the answers. Load [`references/spec-template.md`](references/spec-template.md) and follow its layout: `spec.md` (the standalone decision brief plus the Contents reading contract), `detail/contract.md` (the sections the implementer and auditor consume), and `detail/evidence.md` (the review-time reasoning), every canonical section under its canonical heading in its home file. Keep the brief readable on its own; group the requirements by subsystem/seam and place each success criterion beside the requirement it verifies. Stay at WHAT+WHEN altitude (no file manifest, no exact symbols, no typed code block); the exception is the commands under *Verification approach & commands* in `detail/contract.md`, which are concrete. Record each fact once and reference it by ID (`FR-001`, `SC-001`), section name, or link elsewhere. Set `parent` to the project spec's id, and where a requirement leans on a shared invariant or contract, cite its `INV-NNN` or entity name rather than copying it. If the work item needs a shared fact the project spec does not yet hold (a new invariant, entity, or contract change), **stop**: that is a project-spec amendment (see `grounding-specs`). Get it approved there, then resume; never invent a shared fact locally. Every `< >` placeholder gets a real answer or a justified `N/A — reason`. Set `status: draft`.
5. **Final pass: four self-consistency sweeps.** Apply each good practice everywhere it's warranted, not just where you first thought of it:
   A. **Claim ↔ mechanism.** Every guarantee word (every, only, always, never, cannot) traces to an FR whose check is as strong as the verb; where it's weaker, downgrade the wording. Caveat every instance of a weakness, not just the first.
   B. **Atomicity.** One FR = one verifiable obligation. Split any SHALL that needs more than one pass/fail test.
   C. **Coverage.** Every FR has an SC beside it, or a reason under **Coverage exceptions** in *Known limitations & honest gaps*. No silent blanks.
   D. **Normative vs. contingent.** Move "only if X fails" fallbacks to the **Contingencies** block of *Failure modes* (in `detail/evidence.md`); the requirements list holds only what the release must satisfy and can verify now.
6. **Polish.** Do an editing pass: remove signs of AI-generated writing so the prose reads naturally, and fix ragged line breaks. Readability only: touch no decision, no `SHALL` line, no ID, no header, no fenced block, and leave the frontmatter intact.
7. **The one human gate.** Show the user the spec, the key decisions, and the residual risks. Suggest they have an agent review the spec with a fresh context. Once reviewed, the user should change the spec status to `approved`.

## What you do NOT do

- No implementation. A finished spec is not permission to build it; the human's `approved` is. "It's obvious what to code" and "this saves a round-trip" are the tells that you're stepping over the gate. The spec exists precisely so a reviewer catches what already feels obvious to you. Leave `status: draft`. You never set `approved` on your own say-so; that sign-off is the human's.

## Next step

The spec is drafted at `status: draft`. The next step is `reviewing-specs`: a skeptical pass in fresh context before anyone builds. Offer it. If it returns REVISE, resolve the findings here and re-review. Approval is the human's; once they're satisfied they set `status: approved`. `implementing-specs` waits on that gate; do not start it while the spec reads `draft`.
