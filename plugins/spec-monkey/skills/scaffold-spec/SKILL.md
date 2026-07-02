---
name: scaffold-spec
disable-model-invocation: true
argument-hint: "[what you want to build or change]"
model: sonnet
effort: high
description: "Scaffold a blank spec for the user to write themselves, then critique it. Use when the user wants to author the spec by hand — to think it through and sharpen their engineering — instead of having create-spec do the interrogation and design for them. Creates a lean blank at docs/specs/{slug}/spec.md, then (when they're ready) runs spec-monkey:plan-reviewer and coaches them through the findings. Never writes the spec content for the user."
---

# Scaffold Spec

This is the **write-it-yourself** path. `create-spec` does the thinking for the user; this
skill hands them a blank and critiques what they write. The point is to make them a sharper
engineer. They fill the sections, and the review teaches them where the gaps are.

You do two things: scaffold the blank, then (when the user is ready) critique it. **You never
write the spec content for the user.** Coaching, not ghost-writing.

## Phase 1: Scaffold

- Settle on a short kebab-case `{slug}` for the change; confirm it with the user.
- Copy `reference/blank-spec.md` to `docs/specs/{slug}/spec.md`. Fill only `created` (today) and
  a stable `id`. Leave every other field blank. That's the user's to write.
- Tell the user the file is ready, and what each section asks for, one line each:
  - **Goal**: one verifiable sentence; the delta, tied to an observable outcome.
  - **The request / Why it matters**: the ask in a sentence; the problem and the cost of inaction.
  - **What I'm building**: the behavior, the `FR-NNN` requirements, and what's out of scope.
  - **How it fits**: verified facts about the code, hard constraints, external dependencies.
  - **Data contracts**: the types/signatures the change introduces (or `N/A — reason`).
  - **What could go wrong**: the non-obvious edge cases, each with a decision.
  - **Who's affected**: the blast radius (external, internal, and what's unchanged).
  - **How I know it works**: success criteria, the verification plan, one worked case.
  - **How it ships**: rollout and rollback.
- Remind them: the spec is **design only**, with no file manifest, symbols, or run commands.
  Those go in `tasks.md`, derived by the decomposer after approval.
- Then stop. The user writes. They return when ready.

## Phase 2: Critique (when the user says the spec is ready)

- Dispatch `spec-monkey:plan-reviewer` on the spec. It returns BLOCKING / SHOULD_FIX / SUGGESTIONS.
  (No reference-linter yet: the spec has no file manifest to check. That runs after decomposition.)
- Relay the findings grouped by severity. For each, **explain the principle, not just the fix**:
  why a competent reviewer flags it, so the user learns the pattern. (E.g. "this success criterion
  can't be measured because nothing emits the number, so add the log/counter that would.") That
  teaching is the whole point of this path.
- **Do not rewrite their spec.** Point at what's weak and why; give direction, not finished prose.
  Let them make the fix.
- Re-review after they revise, until the `spec-monkey:plan-reviewer` verdict is APPROVE.

## Phase 3: Decompose & hand off

Once the `spec-monkey:plan-reviewer` verdict is APPROVE and the user approves the spec:

- Set the spec's `status` → `reviewed`.
- Hand it to `spec-monkey:spec-decomposer`. It derives the per-task file manifest and run commands
  from the spec's design plus the codebase, and writes the readable per-task sections into a
  sibling `tasks.md`. Relay its report: the parallel waves, the trade-offs, and any flagged tasks.
- **Cheap existence pass:** dispatch `spec-monkey:reference-linter` on the spec folder. It checks
  every file, symbol, and package the tasks cite. Fix any MISSING / MISLOCATED. Set `status` →
  `decomposed`.
- Then **offer** to run the `execute-spec` skill to implement. If the user accepts, hand off to it;
  otherwise stop here with a reviewed, decomposed spec.
