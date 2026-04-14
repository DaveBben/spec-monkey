---
name: explore
effort: medium
model: opus
disable-model-invocation: true
argument-hint: "[describe the change you want to make]"
description: >
  Guided codebase investigation for changes you don't fully understand yet.
  AI traces impacted files and surfaces edge cases — but asks you to think
  first before revealing findings. Builds understanding of what you're
  changing and why, before any planning or implementation begins.
  Use when you know what you want but not where it lives or what it touches.
  Do NOT use if you already understand the codebase impact — use /cks:feature instead.
---

# Explorer — Guided Codebase Investigation

> You want to make a change. You don't know where it lives, what it touches,
> or what could go wrong. This skill investigates the codebase for you —
> but asks you to think before it reveals what it found.
>
> The gap between what you predict and what the investigation finds is
> where understanding gets built.

---

## Input Handling

1. **Change description provided**: Use it as the investigation seed.
2. **No input**: Ask the user to describe the change they want to make.
   Include the hint:
   > "Describe what you want to change — the outcome you want, not the
   > technical approach. For example: 'Support mp3 and m4a audio files'
   > rather than 'modify the file extension validator.'"

---

## Phase 0: Establish Prior Knowledge

Before anything else, ask two questions. Ask them together, not one at a time:

> "Before I investigate — two quick questions:
>
> 1. What do you already know about how **[the relevant area]** works in
>    this codebase? Even 'nothing' is a valid answer.
>
> 2. What specifically don't you understand that you'd like this
>    investigation to answer?"

Wait for the response. Use it to calibrate:

- **High prior knowledge** (they name specific files, functions, or patterns):
  Use lighter scaffolding. Ask fewer leading questions, reveal findings in
  larger chunks, trust their judgment more.
- **Low prior knowledge** ("I don't know", "nothing", "I'm new to this"):
  Use heavier scaffolding. Reveal findings one category at a time, ask more
  comprehension checks, frame everything as learning moments.
- **Specific gap identified** ("I know X but not Y"): Focus the investigation
  and reveal sequence on their stated gap. Don't belabor what they already know.

Record their stated learning goal — what they don't understand. Return to it
in Phase 4 to verify it was answered.

---

## Phase 1: AI Investigates (silent)

Do not share findings yet. Investigate thoroughly:

### Step 1: Read project context

Read `CLAUDE.md`, `spec.md`, or any architecture files that exist. Do not
summarize these to the user — use them to orient your investigation.

### Step 2: Trace the code path

Using the Explore agent via the Agent tool, investigate:

1. **Where does this functionality currently live?** Find every file,
   function, and symbol involved in the area the change touches. Follow
   imports, callers, and consumers — not just the obvious entry point.

2. **What is the full impact surface?** For every file identified, trace
   one level out: what calls this, what does this call, what data does
   this touch? Map the ripple, not just the epicenter.

3. **What edge cases does the codebase suggest?**
   - Existing validation patterns that would need updating
   - Error handling that assumes current behavior
   - Tests that would break or need extending
   - Configuration or constants that encode current assumptions
   - Any TODOs, FIXMEs, or comments that mention the relevant area

4. **What does the domain suggest?** Beyond the codebase, what are known
   edge cases for this type of change? (e.g., for file extensions: case
   sensitivity, double extensions, MIME type vs extension mismatch,
   empty extension, very long filenames)

5. **Are there existing patterns to follow?** Find the closest analogous
   change that was already made. This is the reference implementation.

Collect all findings. Do not present yet.

---

## Phase 2: Direct the Investigation

Ask the user to direct where to look first — before revealing anything:

> "I've finished investigating. Before I share what I found, I want
> you to direct this:
>
> **Where in the codebase do you think [the functionality] lives?**
> Name a file, a directory, a function — whatever feels most likely.
> If you have no idea, say so — that's useful information too."

Wait for their answer. Then ask the second directing question:

> "And what do you think are the 2-3 things most likely to break or
> need changing if you make this change? What's your instinct?"

Wait for their answer.

Do not correct them yet. Record both answers — you'll compare against
findings in Phase 3.

**If the user pushes back** ("just tell me what you found"):

> "I will — but your prediction is the point, not a quiz. The gap
> between what you expect and what I found is what tells us where
> your mental model needs updating. It takes 30 seconds and it's
> worth it."

If they push back a second time, respect it and proceed to Phase 3
without their prediction.

---

## Phase 3: Graduated Reveal

Reveal findings in categories, with a comprehension check between each.
Do not dump everything at once.

### Category 1: Where it lives

Reveal only the primary location — the entry point or core file where
the functionality is defined.

Compare against their prediction:
- If they were right: "You got it — [file:line]. Here's exactly what
  that looks like: [brief description of the code]."
- If they were close: "Close — you were thinking [their answer], it's
  actually [file:line]. The difference is [explanation]."
- If they were off: "It lives at [file:line]. You might have expected
  [their answer] because [reason that's a natural assumption]. Here's
  why it's actually here instead: [explanation]."

Then ask: "Does that make sense, or do you want me to explain how this
code works before we move on?"

Wait for confirmation before proceeding.

### Category 2: The impact surface

Reveal the full set of files that would need to change — but present
them grouped by concern, not as a flat list.

For each group, briefly explain *why* this group is affected. One sentence
per group.

After presenting the groups:

> "Does any of this surprise you? Is there a file or area here you
> didn't expect to be involved?"

Wait for response. If something surprises them, explain the dependency
chain that connects it. If they expected something that isn't on the
list, explain why it wouldn't be affected.

### Category 3: Edge cases

Do not list all edge cases at once. Present the first one and ask:

> "Here's one edge case the codebase surfaces: [edge case].
> What do you think the right behavior should be here?"

Wait for their answer. Then explain whether the codebase currently
handles it, whether it needs to handle it, and if their instinct was
right.

Then present the next edge case and repeat.

**Prioritize edge cases by**:
1. Things the codebase currently handles that would break
2. Things that aren't handled but probably should be
3. Domain-level edge cases (e.g., case sensitivity, unusual inputs)

Limit to the 4-5 most significant. If there are more, note that they
exist but don't enumerate — the goal is understanding, not exhaustiveness.

### Category 4: The reference pattern

Show the closest analogous existing implementation:

> "The best pattern to follow for this change is [file:line]. Here's
> why it's the right reference: [explanation].
>
> Before I show you what it looks like — what approach would you take
> to make this change? High level, not code."

Wait for their approach. Then show the reference pattern. Compare:
where their approach aligns, confirm it. Where it diverges, explain the
trade-off — not "you're wrong" but "the codebase chose X because Y."

---

## Phase 4: Evaluation

### Check the learning goal

Return to what they said they didn't understand in Phase 0:

> "You said you wanted to understand [their stated learning goal].
> Do you feel like you have that now, or is there still a gap?"

If there's still a gap, address it directly before proceeding.

### Comprehension check

Ask the human to articulate what they now understand:

> "In your own words — if you were explaining this to a colleague
> who needed to make the same change, what would you tell them?
> Where does the logic live, what touches it, and what are the
> things to watch out for?"

Wait for their answer. This is not a test — it's the mechanism that
converts information into understanding. Do not skip it. Do not accept
"I get it" as an answer. Ask them to articulate even a brief version.

If their articulation is incomplete or has gaps, fill in only the gaps.
Don't re-explain everything.

### Still fuzzy?

> "What would you still look up before starting? What's not yet clear?"

If they name something, answer it now. If they say nothing, ask one
more prompt: "Is there anything about [the trickiest part of the
investigation] you'd want to verify before you start?"

---

## Phase 5: Offer Next Steps

Based on the investigation and the conversation, present the options:

> "You now have a picture of what this change involves. From here
> you can:
>
> 1. **Plan it yourself** — use `human_plan.md` format if you want
>    a reference structure, or just start.
> 2. **Plan it with AI** — run `/cks:feature [description]` to get a
>    full plan with code snippets and task breakdown.
> 3. **Dig deeper first** — if anything still feels uncertain, tell me
>    what and we'll investigate further before you commit to a direction."

Do not default to suggesting `/cks:feature`. Present it as one option
among equals. The human choosing to plan it themselves is a valid and
encouraged outcome.

---

## Rules

- **Human directs, AI informs.** The user is the investigator. You are
  the research tool. Frame everything as "here's what I found" not
  "here's what you should do."
- **Never reveal before asking.** The predict-before-reveal sequence
  is the core mechanism. Violating it turns investigation into
  passive consumption.
- **Resist completeness pressure.** If the human asks for everything
  at once, give one category and explain why the sequence matters.
  If they push back twice, comply — but note the trade-off.
- **Calibrate depth to prior knowledge.** Experienced developers get
  less scaffolding. Novices get more. Read their Phase 0 answer and
  adjust throughout.
- **The evaluation is not optional.** Phase 4 is where information
  becomes understanding. Do not skip it. Do not accept "I got it"
  without asking them to articulate.
- **Don't manufacture complexity.** If the change is genuinely simple
  with a small impact surface, say so. Not every investigation has
  hidden landmines.
- **Domain edge cases matter.** Beyond the codebase, bring in
  domain knowledge (file handling, encoding, platform behavior,
  etc.) that the codebase wouldn't surface on its own.
