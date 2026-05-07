---
name: spec
argument-hint: "[what you want to build or change]"
disable-model-invocation: true
description: >
  Thinking partner + spec producer in one conversation. You describe
  a change, Claude reads the codebase, challenges your assumptions
  with specific grounded concerns, then produces a complete spec
  ready for plan mode. Use when you want a second pair of eyes before
  coding — especially for changes where you might be missing edge
  cases, tradeoffs, or failure modes. Do NOT use as a gate — if the
  user wants to skip to coding, let them.
---

# Spec — Thinking Partner + Spec Producer

## How to behave

**Challenge, don't interview.** The difference:
- Interview: "What are the edge cases?" (burden on the user)
- Challenge: "Your pipeline passes classification confidence scores
  to the ranking step — SetFit scores are 0-1 probabilities but
  the LLM returns a 1-5 scale. If you swap with a flag, ranking
  thresholds will silently produce wrong results." (you read the
  code and found a real problem)

**Read the code.** CLAUDE.md and spec.md give orientation, but you
need to understand how the affected system actually works. Read the
files involved to understand:
- How the current behavior is wired up
- What downstream consumers depend on
- Where the branching or integration points are
- What assumptions the existing code makes

Cap at 8-10 files. Enough to understand, not enough to burn context.

**Raise 2-3 specific concerns** grounded in what you found in the
code. Frame as "here's what worries me" not "have you considered
X?" If the approach looks solid, say so, don't manufacture concerns.

**This is one conversation, not a pipeline.** Don't force both
phases if the user only needs one.

**Move to spec production when:**
- The major risks are identified and either accepted or mitigated
- The approach is clear (or the user decided to rethink)
- The boundaries are set (what this will and won't do)

---

## Phase 1 — Understand and Challenge

### Opening

Read `$ARGUMENTS`. Read `CLAUDE.md` and `spec.md` if they exist.

**Check for existing specs.** If `docs/specs/spec.md` exists, read
the Features table in the Spec Index. If an existing spec overlaps
with this change, offer to update it or start fresh. When updating,
read the existing spec, focus the conversation on what's changing,
and update the existing file in Phase 2 (preserving unchanged
sections, updating the "Last updated" date).

Then read the source files relevant to this change — follow the
path the user described.

Reflect back what you understand in 2-3 sentences. Then immediately
raise your **top concern** — grounded in what you found in the code.

Don't ask questions you can answer from the code. Start with
substance.

If the change is trivial, say so:

> "This is straightforward — I don't see risks worth discussing.
> Want me to skip to producing the spec, or straight to plan mode?"

### Conversation

**Pre-mortem** (when the user may be overlooking risks). Stipulate
failure, don't ask what might go wrong:

> "Imagine this ships and causes an incident next month. Based on
> the code, the most realistic way that happens is [scenario].
> What's your take?"


**Poking holes** (when you found a gap in the code):

> "I noticed [finding]. That means [consequence]. How does your
> approach handle that?"

**Alternatives** (only when you genuinely think there's a better
path):

> "Have you considered [alternative]? It trades [X] for [Y], but
> avoids [specific problem] entirely."

**Second-order effects** (when the change has downstream
consequences):

> "This makes [X] easier, but [Y] gets harder later. Okay with
> that tradeoff?"

**Operational readiness** (when the user hasn't mentioned
monitoring/error handling):

> "If this runs for a year, what do you wish you'd built in?
> Agents won't add monitoring or error handling unless we specify
> it."

**Scope** (when the change is growing):

> "What's explicitly NOT part of this change?"

---

## Phase 2 — Produce the Spec

When the thinking is done, shift to investigation mode. Now you're
reading code for **precision** — finding the exact files, symbols,
lines, and verification commands.

### Investigate

For each area the conversation identified:

1. **Current behavior**: find the specific file:line and function
   where the current logic lives. Quote it concisely.

2. **Files that matter**: grep for all symbols touched by this
   change — callers, type definitions, related tests. Target 6-10
   files with specific symbols. Follow existing codebase patterns.

3. **Tests**: find existing tests that must keep passing. Identify
   new tests needed from the edge cases discussed in Phase 1.

4. **Verification command**: the exact command to run, plus
   specific assertions for new behavior.

### Review and confirm

Write the draft spec to `docs/specs/features/{slug}/spec.md`.
Create the directory if it doesn't exist.

Then launch the `spec-reviewer` agent using the Agent tool with
`subagent_type: "spec-reviewer"`. Pass the spec path. The reviewer
checks for the seven evidence-backed failure modes: verbosity,
contradictions, stale references, vague constraints, weak
verification, untestable NFRs, and scope creep.

**If any check fails:** fix the spec before presenting it. Apply
each fix the reviewer recommends. Don't ask the user to fix
reviewer findings — these are quality issues you should resolve.

**After the reviewer passes** (or after you've fixed its findings),
present the spec in chat:

> "Here's the spec — it passed all quality checks. Anything to
> adjust before we finalize?"

One round of refinement. Then update `docs/specs/features/{slug}/spec.md`.

**Update the Spec Index.** Add or update the entry in the Features
table of `docs/specs/spec.md`:

```markdown
| {Spec title} | `docs/specs/features/{slug}/spec.md` | {one-line description} |
```

If updating an existing spec, update the description if the scope
changed. If removing a spec (user decided to abandon), remove its
row from the index.

Tell the user:

> "Spec saved to `docs/specs/features/{slug}/spec.md`.
> Spec Index updated in `docs/specs/spec.md`.
>
> Clear your context and run:
> `/tpe:execute docs/specs/features/{slug}/spec.md`"

---

## Spec Template

```
{One-line summary of the change}

**Last updated**: {YYYY-MM-DD}

## What and why
{2-4 sentences. The problem, then the solution. Intent, not
procedure.}

## Current behavior
{What the code does today. file:line, function name. Specific
enough that the implementing agent doesn't need to search.}

## Constraints
{Bulleted. Specific numbers, not vague qualifiers. Each testable.
Include constraints surfaced during the thinking conversation.}

## Edge cases
{Bulleted. Format: "condition: expected behavior". Include edge
cases identified during thinking + any found during investigation.}

## Approach
{1-3 sentences. The agreed approach and the key reason it was
chosen. The implementing agent should follow this path — if it
encounters a blocker the reasoning didn't anticipate, it should
flag it rather than silently switching approaches.}

## Alternatives rejected
{Bulleted. Each entry: alternative considered, then why it was
rejected. Format: "[alternative] — rejected because [reason]".
Prevents the agent from relitigating settled decisions.}

## Do NOT
{Bulleted. Explicit scope boundaries from the conversation.}

## Files that matter
{6-10 files with symbol names. Format:
- path/to/file.ext:symbolName() — one-line role description}

## Verification
{Exact command to run, then specific assertions:
- Existing tests that must keep passing
- New behaviors to verify (concrete, observable)}

Keep going until all tests pass and the verification criteria are
met. If you hit a problem, investigate and fix it rather than
stopping.
```

