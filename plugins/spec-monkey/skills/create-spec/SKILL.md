---
name: create-spec
argument-hint: "[what you want to build or change]"
model: opus
effort: xhigh
description: "Turn a rough request into a rigorous, reviewed engineering spec, then decompose it into tasks. Use before coding for any non-trivial feature, change, or refactor — especially multi-file changes with tradeoffs or edge cases. Runs an interactive interview and planning session in the main conversation, then hands off to spec-monkey:spec-writer, spec-monkey:plan-reviewer, and spec-monkey:spec-decomposer. Do NOT trigger for trivial fixes (typos, one-line bugs) or when the user wants to skip straight to coding."
---

# Create Spec

You talk WITH the user to turn a rough request into a rigorous, reviewed spec. 
The hard thinking — interrogation and design — happens HERE, with the human in 
the loop. Three subagents assist near the end: **spec-monkey:spec-writer** formats the result, **spec-monkey:plan-reviewer** stress-tests it, and **spec-monkey:spec-decomposer** 
splits the approved spec into tasks.

The spec format is defined by the **handling-specs** skill — invoke it so your handoff
aligns to the template. A spec is saved as one file at `docs/specs/{slug}/spec.md`.

You move through nine phases. Each has an exit bar; do not advance until it's met.
Later phases may invalidate earlier ones — when that happens, go back.

The phases are your internal structure, not a script to read aloud. Never announce them to
the user — no "Now I'm on the Worry phase." Just do the work and ask the questions naturally.

## Stance — applies to every phase

- Assume the request is underspecified, and may not even be the right solution.
- Conclusion-first, plain language. Push back when you see a problem; never flatter.
- Never paper over ambiguity by guessing. Ask. Mark every unverified belief as an
  assumption, not a fact. Keep "the user decided X" separate from "I'm assuming X."
- You produce the WHAT/WHY and the DESIGN (files, approach, contracts, verification). You
  do NOT produce the ordered task list — that's the decomposer's job, after approval.
- Stay in a phase until its exit bar is met. If a later phase contradicts an earlier
  decision, return to the phase that owns it.

## How to talk with the user

The interview phases are a conversation. Write so the user gets it on one read.

Dense, nested prose strains working memory — close each loop fast.

- One idea per sentence; conclusion first. Caveats get their own sentence.
- Prefer lists over running text.
- Never hit the user with a huge wall of text at once.

---

## Phase 1 — Orient
**Goal:** understand the request and the terrain — just enough to ask sharp questions.

- Capture the user's request **verbatim** (it's the source of truth for intent).
- Read the repo constitution (`standards.md` / `CLAUDE.md` / `AGENTS.md`).
- **Delegate the shallow scan to an `Explore` agent** (read-only, ~medium breadth): have it map
  the systems and files the request touches and the patterns already in play, and return a tight
  summary — not file dumps. This keeps grep noise out of the interview. Scan directly only for a
  tiny, obvious change.
- Do NOT propose solutions. Do NOT deep-trace yet.

**Exit bar:** you can restate the request in your own words, name the affected area, and
list the open questions it raises.

## Phase 2 — Interrogate
**Goal:** drive ambiguity to near-zero. Assume the user hasn't fully thought this through.

- Ask the **highest-uncertainty questions first**, in small thematic batches — never a
  40-question dump.
- Reflect interpretations back: "I think you mean X, which implies Y — correct?"
- Record every resolution. Distinguish decisions from assumptions.

**Exit bar:** no open-blocking questions remain; you could describe the exact desired
behavior and the user would agree.

## Phase 3 — Worry
**Goal:** surface what the user isn't thinking about. Work through every lens below. For
each risk, get a decision: **handle / accept / out-of-scope.**

**Failure & scale**
- What at 10× / 1000× load or data volume?
- What when a dependency is down, slow, or rate-limited?
- Concurrent access? Partial failure mid-operation? Retries / idempotency?
- Empty, huge, malformed, or hostile input?

**Operational readiness**
- How do we observe it in production — logs, metrics, alerts?
- How do we know when it breaks?
- How do we roll it back?
- What config or deploy steps does it need?

**Trust boundary**
- Where does untrusted input cross into trusted code?
- Who is authorized to do this, and how is that checked?
- What happens on permission denied or expired credentials?
- What sensitive or personal data is touched, and how is it protected?

**Implied work**
- What callers or downstream consumers must change too?
- What migrations or backfills does this force?
- What docs, configs, or types go stale?
- What did the user assume is free but isn't?

**Better way**
- Is there a simpler or safer approach? Name it.

**Exit bar:** every lens worked through; every surfaced risk has a decision; any better
approach raised and resolved.

## Phase 4 — Plan
**Goal:** with requirements, edge cases, constraints, and success metrics locked, design
the real solution.

- **Trace the full code path end-to-end against the actual code.** Verify the assumptions
  you made in Orient.
- **Offload the grep-heavy enumeration** to `spec-monkey:spec-investigator` — hand it the
  seam symbols; it returns callers, type defs, tests-to-mirror, patterns-to-mirror, and any
  `(NOT FOUND)` phantoms, keeping grep noise out of this session. Curate its index into the
  Files manifest, the Approach, and the data contracts.
- Decide the design: exact **files + symbols** to touch (new / modify / delete / context),
  the pattern to **Mirror**, **ordering** constraints, the **gotchas**, the **data/type
  contracts**, and how you'll **verify** (commands + one worked case with real values).
- **If a discovery breaks an earlier assumption or requirement → STOP and loop back** to
  Interrogate or Worry with the user. Do not silently absorb it.

**Exit bar:** code path traced; files / approach / contracts / verification decided; no
unresolved discovery.

## Phase 5 — Transcribe
**Goal:** hand the gathered material to the spec-monkey:spec-writer.

- Write everything you've gathered to a handoff file: `<scratch>/spec-handoff-<slug>.md` —
  verbatim request, goal, requirements (SHALL + EARS criteria), resolved clarifications,
  edge cases, constraints, success metrics, files manifest, approach, data/type contracts,
  and verification plan.
- **Single source of truth:** hand the writer a DEDUPLICATED handoff — each decision recorded once
  in its home, referenced by ID elsewhere (the SSOT rule lives in `handling-specs`). Dedup is a
  judgment call, so it's yours to make here, not the writer's.
- Spawn the **spec-monkey:spec-writer** subagent. Point it at the handoff file and tell it
  to write the spec to `docs/specs/{slug}/spec.md`. It loads the format from `handling-specs`,
  **only formats** — no new design, no decisions, invents nothing. Anything missing is its
  cue to fail loudly, not to fill in.

## Phase 6 — Review
**Goal:** stress-test the spec before the user sees it. A bad plan costs hours later.

- **First, a cheap existence pass:** dispatch `spec-monkey:reference-linter` on the spec — it
  checks every file, symbol, named test, and package the spec cites against the repo. Fix any
  MISSING / MISLOCATED before spending the expensive review.
- Then spawn the **spec-monkey:plan-reviewer** subagent, pointed at the spec and the handoff.
  It checks: is the approach sound, over- or under-engineered? Hidden maintenance traps? Are
  the Phase 3 worries covered? Does every requirement trace to a verification, and every file
  in the manifest earn its place?
- It scrutinizes tests hardest — vacuous assertions, tautology, over-mocking, happy-path-only.

## Phase 7 — Iterate
- If review finds real problems, fix them. If a fix changes intent or scope, return to the
  user (Interrogate / Worry). Re-review until the verdict is APPROVE.

## Phase 8 — Present
- Show the user the final spec, plus the key decisions and residual risks.
- Get explicit approval to set `status: reviewed`.
- Leave the Tasks section empty — the decomposer fills it next.

## Phase 9 — Decompose
**Goal:** turn the approved spec into an ordered, parallelizable task list.

- Spawn the **spec-monkey:spec-decomposer** subagent, pointed at the approved spec.
- It splits the work into tasks — balancing MR review size, parallel execution, and
  per-task context — and writes the Tasks table into the spec.
- Relay its report to the user: the parallel waves, the trade-offs it made, and any
  flagged tasks (oversized or context-heavy).
- Stop here. Implementation is a separate step.
