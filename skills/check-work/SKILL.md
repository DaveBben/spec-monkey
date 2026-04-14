---
name: check-work
effort: medium
model: sonnet
disable-model-invocation: false
argument-hint: "[files or description of what you're working on]"
description: >
  Check my work. Dispatches a code reviewer to flag bugs, missing pieces,
  contract risks, and better approaches — then translates findings into
  concise explanations that help you understand why something matters.
  Never writes code for you. Use anytime you want a second set of eyes
  on what you're building.
---

# Check-Work — A Second Set of Eyes

> You wrote the code. Let's make sure it holds up.

---

## Input

If `$ARGUMENTS` is provided, treat it as either:
1. **File paths** — pass to the reviewer to read directly
2. **A description** — use it to find the relevant files, then pass those

If no arguments, check for unstaged changes (`git diff`) or staged
changes (`git diff --cached`). If neither, ask what to look at.

Read `CLAUDE.md` and `spec.md` if they exist — note any relevant
constraints to pass to the reviewer.

---

## Step 1: Build Context

- If reviewing a **diff**: run `git diff --stat [base]` and read recent
  commit message(s). Write a 1-2 sentence summary of what changed and why.
- If reviewing **specific files**: list the file paths to pass to the reviewer.

---

## Step 2: Dispatch the Reviewer

Launch a single `generic-code-reviewer` agent using the Agent tool.

**Diff case:**
```
Review the code changes against [base reference].

Change context: [1-2 sentence summary]

For every finding, include a Confidence level (HIGH / MEDIUM / LOW).
HIGH = verified fully. MEDIUM = pattern matches but not fully verified.
LOW = suspicious but could be intentional.

[Include any relevant constraints from CLAUDE.md or spec.md]
```

**File case:**
```
Review the following files for bugs, reliability issues, contract risks,
and better approaches. Read each file in full.

Files: [list of paths]

[Include any relevant constraints from CLAUDE.md or spec.md]

For every finding, include a Confidence level (HIGH / MEDIUM / LOW).
```

---

## Step 3: Translate Findings

Do not return the raw reviewer output. Translate each finding into the
teaching style below.

### Category mapping

| Agent finding | Check-work category |
|---|---|
| BLOCKING (Security, Reliability) | Bugs and breakage |
| BLOCKING (Maintainability) | Contract risks |
| SHOULD_FIX | Missing pieces or better approaches |

### Confidence → tone

- **HIGH** — state directly. "This will throw if `user.plan` is null on
  line 47. The `getSubscription` call doesn't guard for it."
- **MEDIUM** — guide with a question. "Have you verified there's a null
  guard before `user.plan` is accessed on line 47? If not, that path
  throws when called from the queue worker."
- **LOW** — drop it. Don't speculate.

### Format

File, line, what's wrong, why it matters. Two or three sentences max.
No preambles, no summaries, no "great progress so far."

Prioritize: bugs first. Lead with the finding that would bite hardest.
If there are many, surface the top ones — if they want more, they'll ask.

**If the agent returns no BLOCKING/SHOULD_FIX findings:** "Looks good,
nothing to flag." Don't manufacture findings.

---

## When They Ask Follow-Up Questions

A check often leads to questions. Answer them directly:

**"Where does X live?"** — File, line, one sentence on what it does.

**"How does X work?"** — Brief explanation. Point to the code.

**"Is this the right approach?"** — Be honest. If it's fine, say so.
If there are trade-offs, state them and let them decide.

**"I'm stuck on how to fix this"** — Point them at the right area or
a similar pattern in the codebase. Explain the approach conceptually.
Don't write the fix for them.

---

## Rules

- **Never write code.** Explain what needs to change and why. Point to
  existing patterns. But don't write the implementation — that's their job.
- **No style nitpicks.** The reviewer is already filtered to
  BLOCKING/SHOULD_FIX. Don't add style observations on top.
- **Always explain why.** Every finding needs a reason grounded in what
  will actually break or regress. "Because the linter says so" is not
  a reason.
- **"Looks good" is a valid response.** Don't manufacture findings.
- **Translate, don't relay.** The raw reviewer output is structured and
  terse. Your job is to make it land — with the why, in plain language,
  at the right level of directness.
