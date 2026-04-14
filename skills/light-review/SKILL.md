---
name: light-review
description: >
  Use proactively when the user requests a review of their code changes. Launches a single-agent code review covering security, reliability, performance, and maintainability in one pass. Appropriate for any code change — cost-efficient alternative to /cks:deep-review. For a thorough 4-agent Opus review, use /cks:deep-review instead. Do NOT use for reviewing existing code without a diff context.
argument-hint: "<base-branch-or-commit>"
effort: medium
disable-model-invocation: false
---

# Fast Code Review

Dispatch a single `generic-code-reviewer` agent using the Agent tool to perform a rapid review of
code changes. Covers all four review dimensions in one pass, focusing only on findings that must
be fixed.

## Workflow

### Step 1: Determine the diff base

- If `$ARGUMENTS` provided → use as base branch or commit SHA
- If nothing provided → check for staged changes (`git diff --cached`)
- If nothing staged → diff against the main branch (`main` or `master`)

### Step 2: Build change context

Before dispatching the agent, build a brief change context summary:

1. Run `git diff --stat [base]` to get files changed and line counts
2. Read the most recent commit message(s) on the branch to understand intent
3. Write a 1-2 sentence summary: what changed and why

### Step 3: Dispatch the agent

Launch a single `generic-code-reviewer` agent using the Agent tool with:

```
Review the code changes against [base reference]. [Base reference] is the base branch
or commit to diff against.

Change context: [1-2 sentence summary from Step 2]

For every finding, include a Confidence level (HIGH / MEDIUM / LOW) indicating how
certain you are this is a real issue. HIGH = you verified it fully. MEDIUM = pattern
matches but not fully verified. LOW = suspicious but could be intentional.
```

### Step 4: Return the report

Return the agent's report directly — no consolidation needed (single agent).
