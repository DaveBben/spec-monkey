---
name: example
description: "Quick-reference cheatsheet that shows how to do something with a concise example or definition. Use when the user asks 'how do I...', 'what is...', 'show me...', 'syntax for...', 'command for...', or wants a quick command/concept/cheatsheet reference. Do NOT use for problems requiring deep reasoning, code review, implementation tasks, planning, or file modifications"
user-invocable: true
disable-model-invocation: false
argument-hint: "<question>"
model: haiku
effort: low
allowed-tools: []
---

# Quick Reference

Generate a concise cheatsheet-style answer for the user's question. Seeing a
concrete example first, then understanding it, builds the strongest mental model.

## What to do

Answer the following question directly in cheatsheet format — code/command first,
then one sentence of context.

**Query:** $ARGUMENTS

## Response format

- Lead with the code snippet, command, or definition
- Follow with at most one sentence of context or explanation
- Do not add preamble, commentary, or follow-up suggestions
- The answer should stand alone as a quick-reference card

## Gotchas

- Brevity is the point — do not pad the response.
- If the query is ambiguous, answer the most common interpretation rather than
  asking for clarification (cheatsheets favor speed over precision).
- If your initial answer exceeds ~10 lines, trim it to the essential
  command or example plus one sentence of context.
