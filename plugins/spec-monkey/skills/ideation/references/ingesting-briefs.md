# Ingesting an existing brief or PRD

The interview is not the only door. When the user already has a written brief, PRD, ticket, design doc, or page of answers, don't re-interview from scratch — that friction makes people quit before the payoff. Read the document, map what it settles, interview only the gaps. This is a fast path *into* ideation, not around it: the risk lenses and approach comparison still run.

## When to reach for this

The user hands you, or points you at, a document that already carries some of the thinking: "here's the PRD", "I wrote up what I want in `notes.md`", a pasted spec from another tool, a filled-in answer list. If they only have a sentence, run the normal interview; there is nothing to ingest.

## The moves

1. **Read the whole thing first.** Before a single question, read it end to end. Look for what it decides, what it assumes, and what it leaves open — not for a place to start asking.

2. **Map it onto the design structure, don't transcribe it.** For each part of the reasoning — the ask, current-state facts, drivers, risks, candidate approaches, decisions — find what the document supplies and restate it in spec terms. A PRD's prose becomes a *Driver*; a stated constraint, a fact under *What's true today*; a chosen approach, a *Decision to sign off*. Copying the document's words isn't the work; extracting its decisions is.

3. **Separate what it settles from what you inferred.** What the document states is a decision. What you filled in to make it make sense is an **assumption** — mark it so, and never launder an inference into a fact. This keeps a confident-sounding brief from smuggling unexamined choices into a signed contract. "The PRD implies X" is an assumption to confirm, not a decision to record.

4. **Reflect the extraction back once, in a batch.** Show the user your read: "here is what I take you to have **decided**, what I am **assuming** to fill the gaps, and what the document leaves **open**." This one pass replaces the one-question-at-a-time interview for everything the document already answers — cheaper precisely because the answers exist. Correct mistakes, promote or kill the assumptions, then move on.

5. **Interview only the residual gaps.** Run the normal interview only over what the document left open — highest-uncertainty questions first, one at a time, reflecting each answer back. Skip every question the brief answers. A ten-question interview may collapse to two.

## What a brief does NOT let you skip

- **The risk lenses.** A brief rarely works all five (failure & scale, operational readiness, trust boundary, implied work, better way). Work the ones it skipped; each surfaced risk still gets HANDLE / ACCEPT / OUT-OF-SCOPE. A PRD that lists features has almost never worked the trust boundary.
- **The approach comparison.** A brief that asserts one approach with no alternatives is a decision to *ratify*, not a reason to skip weighing it. Name the real alternatives, lay out the tradeoff, and put the choice — the brief's or a better one — to the human. If the brief's approach is right, say why; if the comparison changes it, that's the interview earning its keep.
- **The one-decision gate.** A big PRD often bundles several decisions with distinct owners or lifecycles. Split it into sequenced children like any other over-large ask; the brief buys no exemption.
- **The constitution's house rules.** The brief's claims still bind to the repo constitution (`standards.md` / `CLAUDE.md` / `AGENTS.md`). A brief that contradicts a house rule or a shared convention is a conflict to surface, not a local exception to grant.

## Altitude

A PRD or design doc may name files, symbols, or paste code — that is the author's HOW, useful context but not the spec's content. Ingest the *intent* behind it; leave the file manifest and the typed code to the implementer. The spec you compose stays WHAT + WHEN even when the source document didn't.
