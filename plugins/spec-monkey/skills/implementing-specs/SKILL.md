---
name: implementing-specs
version: "1.4.0"
description: "Implement an engineering spec end-to-end. Build the code that satisfies its requirements and verify it against the spec's own success criteria and commands. Use when the user provides you a spec to implement. Do NOT use to author or review a spec or for a change with no approved spec."
license: MIT
compatibility: any-agent
---

# Implementing Specs

You take an approved spec and build it. The spec says WHAT to do and WHEN; the HOW is yours. You work that out from the spec and the real code.

## Stance

- Build what the spec says. Don't add scope it didn't ask for, and don't drop a requirement.
- **Uphold the project spec.** If the spec names a `parent`, the project spec's cited `INV-NNN` and shared contracts bind the build as firmly as the spec's own FRs. Never break an invariant to satisfy a local requirement; if you can't have both, stop and raise it.
- If the spec is wrong, impossible, or missing something load-bearing, stop and raise it with the human. Don't silently reinterpret it.
- You are not done until verification passes, every FR and non-functional bound is met, and no cited `INV-NNN` is violated. "Implemented" is a claim; the verification you ran this session is the proof. Don't make the claim without the proof.

## Workflow

Do the work; don't narrate the steps.

1. **Find the spec.** Locate the approved spec folder at `docs/specs/{slug}/` (its `spec.md` `status` reads `approved`). Read `spec.md` and `detail/contract.md` (together they are the full build contract): the requirements (`FR-NNN`) and their success criteria (`SC-NNN`), *When it happens* (triggers and ordering), the *Data & interface contract*, the scope and *Out of scope* lines, and *Verification approach & commands*. `detail/evidence.md` is review-time reasoning; open it only when the contract leaves a decision unclear. (A spec written under format 1.0.0 spreads these sections one-per-file in `detail/`; locate them by heading and read them all.) If `spec.md`'s frontmatter names a `parent`, read the project spec (`docs/specs/project/spec.md`) too: the `INV-NNN` and shared contracts it cites bind this build.

2. **Plan the build.** Work the implementation out from the spec and the code. Find the seams the change touches: the callers, the types, the tests. Let *When it happens* set the order. How you break the work up is up to you: steps, tasks, or independent non-conflicting pieces run in parallel.

3. **Build it.** Implement until every `FR-NNN` holds. Respect the constraints, the data and interface contracts, and the ordering. Stay inside scope, and build nothing on the *Out of scope* list. Mirror the patterns already in the codebase rather than inventing new ones.

4. **Verify.** Run the spec's verification from *Verification approach & commands* in this session, and read the output before you trust it. Every success criterion must hold, the worked case must produce the exact values it states, and every cited `INV-NNN` must still hold in the built code. A pass you remember from an earlier run, or expect because the code should work, is not one; run it now. When a check fails, fix the code, not the check: loosening an assertion, skipping the case, or relaxing a bound to reach green fakes the result instead of meeting it.

5. **Review the change.** Read your own diff critically against the spec: does every requirement trace to a real change, are the edge cases handled, are the tests honest (no vacuous assertions, no over-mocking that leaves the real path untested)?

6. **Finalize.** Set the spec's `status` to `implemented` only once the checks passed in this session; the status records what you verified, not what you expect to hold. Then commit with a message that says what changed and why, and points back to the spec.

## Optional: subagent-orchestrated build

The workflow above runs in one context and is the default; it works in any harness. If your harness can dispatch subagents (fresh agents with isolated context you task and read results back from), and the spec's *Requirements & success criteria* splits into three or more FR-groups, you may run the build as an orchestration instead: a fresh implementer per FR-group, a review gate after each, a fix loop until clean, and a progress ledger that survives compaction. It raises quality on larger specs by giving each group a clean context and its own gate; it costs more invocations, so it earns its keep only past two or three groups.

Use it only when both hold: your harness dispatches subagents, and the spec has three or more FR-groups. Otherwise run the workflow above. Never require a subagent; the single-agent path is always sufficient.

The mechanics (per-group dispatch, the three-verdict gate of spec compliance, code quality, and cited `INV-NNN`, the fix loop, the file handoffs, the `.spec-monkey/progress.md` ledger, and model selection) live in [`references/subagent-mode.md`](references/subagent-mode.md). Read it before you orchestrate.

## What you do NOT do

- No scope creep. If you find the spec needs to change, raise it; don't quietly expand the work. If the build needs a shared fact the project spec lacks (a new invariant, entity, or contract change), that is a project-spec amendment: stop and raise it, don't invent it locally.
- No authoring or reviewing the spec itself.

## Next step

With `status: implemented` and the change committed, the build is done but unproven by a second pass. The next step is `auditing-specs`: an independent trace of every requirement to code, with the spec's own verification re-run. Offer it. This is separate from your own review in step 5; a fresh context catches what the builder's does not.
