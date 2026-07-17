# spec-monkey

A set of portable skills for spec-driven development.

A spec is a **design contract**. It describes what to build, why it's needed, what constrains the solution, 
and how you know it works. It's so a reviewer can approve the plan before any code exists. 

**When it earns its keep.** Reach for the full flow on work where a missed requirement is expensive:
complex or multi-seam features, changes a team has to agree on, or long agent runs that must survive a lost
context. Skip it on the small stuff — a one-line fix doesn't need a spec, and `ideation` triages a trivial
item onto a fast lane rather than marching it through the full flow. The ceremony scales to the risk; it
isn't waterfall you pay on every change.

### What it costs, and when to skip it

The flow is not free, and pretending otherwise is how spec tools earn the "technical masturbation" charge.
Reach for it when a missed requirement is expensive; skip it when it isn't.

- **The interviews cost tokens up front.** Ideation and writing each interview you, one question
  at a time. On a complex, multi-seam change that conversation surfaces requirements you'd have missed. On a
  solo throwaway it's overhead — take the trivial lane, or write no spec at all.
- **WHAT-only specs raise the implementer floor.** Because the spec never carries the HOW, the implementer
  re-derives it from the spec and the code each build. That keeps the spec from rotting when the code moves,
  but it means you can't hand the build to the cheapest model the way a plan with the code written in it can;
  mid-tier is the effective floor. The trade buys altitude-stable specs at a higher per-build model cost.
  The design's *Approach* carries the high-level shape into the build, so the implementer re-derives the
  detail, not the intent — but the spec itself still carries no HOW. (If you want a
  fully HOW-carrying plan for cheap execution, see [interop](docs/interop.md).)
- **The portable path costs context hops.** The review and the audit are worth more in a fresh context. On a
  harness without subagents that's a manual new session (a `/clear`, on harnesses that have it) for each — a few hops per feature. A
  subagent-capable harness and the optional hook fold those back in.
- **When it costs more than it saves:** a one- or two-line change; a spike where the design is discovered by
  building, not specified up front; a solo weekend project with no shared architecture. The mid-2026 SDD
  consensus agrees the full ceremony is overkill there.

---

### Installing

**Claude Code (plugin):**

```bash
/plugin marketplace add DaveBben/spec-monkey
/plugin install spec-monkey@spec-monkey
```

Skills are namespaced under `spec-monkey:`, so you get `/spec-monkey:ideation`,
`/spec-monkey:reviewing-designs`, `/spec-monkey:writing-specs`, `/spec-monkey:reviewing-specs`,
`/spec-monkey:implementing-specs`, and `/spec-monkey:auditing-specs`.

**Optional turnkey activation (Claude Code).** Out of the box you start the flow by invoking
`/spec-monkey:ideation`. If you'd rather have "let's build X" reach for it automatically —
the way an auto-triggering assistant does — run the shipped one-command installer for the optional
SessionStart hook:

```bash
hooks/install-hook.sh --user     # or drop --user for this project only
```

It merges one hook entry into your Claude Code settings idempotently (no hand-typed paths), and it is
reversible — delete the entry to undo. The hook stays **off by default and outside the plugin**, so the
skills themselves depend on nothing and remain portable to any harness. Details in [`hooks/`](hooks/).

**Any agent (Skills CLI):**

```bash
npx skills add DaveBben/spec-monkey
```

Installs all six skills into whatever skills-compatible agent you have — Claude Code, Cursor,
Codex, opencode, and others. Target one explicitly with `-a`, e.g. `npx skills add DaveBben/spec-monkey -a claude-code`.

**Other harnesses (adapter layer).** The skills are the same everywhere; what varies per harness is a thin
adapter: a manifest that points at the skills (`.codex-plugin/`, `.cursor-plugin/` at the repo root), a
tool-mapping file that translates the skills' action words into that harness's real tools
([`docs/harness-tools/`](docs/harness-tools/)), and the optional session bootstrap in [`hooks/`](hooks/) —
one env-sniffing script that emits the right hook JSON for Claude Code, Cursor, or the SDK standard, pointing
"let's build X" at `ideation`. The Cursor adapter wires the bootstrap through its manifest; Codex needs none
(skills surface natively). To add a harness, see [`docs/porting-to-a-new-harness.md`](docs/porting-to-a-new-harness.md).

**Test locally:**

```bash
claude --plugin-dir ./plugins/spec-monkey
```

### The six skills

| Skill | What it does | Produces |
|---|---|---|
| `ideation` | The entry point and thinking phase: an interactive interview that works an ask through before its spec is written. Triages the ask (a one-liner it just does; a trivial slice it hands straight to writing; a large ask it decomposes into several linked work items), explores the landscape, weighs 2-3 approaches with their costs, and converges on a design — the *Approach*, the failure modes, and a high-level verification strategy. A gated, reviewable document. | `docs/specs/{slug}/detail/design.md` (`kind: design`) — one per work item |
| `reviewing-designs` | A skeptical, report-only critique of the **design** before a contract is written: soundness of the approach, grounding against the real code, risk-lens coverage, undefended decisions, and whether the verification strategy is credible. Returns APPROVE / REVISE. | A review report (never edits the design) |
| `writing-specs` | Composes the formal work-item spec from the approved design: turns it into the decision brief plus the binding FR/SC contract, the data and interface contract, the timing, and the verification commands. Reads the design rather than re-interviewing. Gates on the human approving the spec. | `docs/specs/{slug}/` — `spec.md` plus `detail/contract.md` (the default; a user-named path wins) |
| `reviewing-specs` | A skeptical, report-only critique of the **contract**: the self-consistency sweep, traceability, weak or vacuous success criteria, whether the verification is trustworthy, and any HOW that leaked past altitude. Returns findings + an APPROVE / REVISE verdict. | A review report (never edits the spec) |
| `implementing-specs` | Takes an approved spec and builds it: works out the HOW from the spec and the real code, building one FR-group slice at a time, test-first, then verifies against the spec's own success criteria and commands. Stops before pushing or opening a PR. | Working code + a commit that points back to the spec |
| `auditing-specs` | Audits a finished implementation against the approved spec: traces every requirement to code, runs the spec's own verification, and confirms the success criteria, data contracts, constraints, and scope boundaries actually hold. Report-only. | A COMPLIANT / NON_COMPLIANT report with evidence |

The flow is `ideation → reviewing-designs → writing-specs → reviewing-specs → implementing-specs → auditing-specs`.
Each skill ends by naming and offering the next, while the human still holds the approval gates; nothing
auto-executes. There is no orchestrator: the `## Next step` handoffs drive it. `ideation` is where you
start — it triages the ask, and on a large ask decomposes it into one design per work item, so you run the
flow per work item.

#### `ideation`

You describe a change; the skill interviews you to think it through before any binding requirement is
written. As the flow's entry point it **triages the ask first**: a one- or two-line change with nothing to
sign it just makes; a trivial slice with a contract still worth signing it hands straight to `writing-specs`
for a light spec; one work item it designs; a **large** ask (several independent decisions, more than a
reviewer reads at once) it **decomposes** into one design per work item, linked by `depends_on` (a hard
ship-order) and `relates_to` (a soft sibling link). For a work item it works the ask, orients in the code,
runs the five risk lenses, and weighs 2-3 approaches with their tradeoffs and a recommendation. The output
is the reasoning: `detail/design.md` plus the brief's *Drivers* and *Decisions to sign off*.

- **Read the constitution.** Read the repo constitution (`standards.md` / `CLAUDE.md` / `AGENTS.md`) and
  scan the touched area just enough to ask sharp questions.
- **Interrogate.** Drive ambiguity to near-zero, highest-uncertainty questions first,
  reflecting interpretations back.
- **Worry.** Work five risk lenses (failure & scale, operational readiness, trust boundary,
  implied work, better way); each surfaced risk gets HANDLE / ACCEPT / OUT-OF-SCOPE.
- **Weigh approaches.** Name 2-3 real alternatives, lay out their tradeoffs, and recommend one
  with a reason; the choice lands in *Decisions to sign off*.

#### `writing-specs`

You hand it the shaped reasoning; the skill composes the contract onto it. It writes `spec.md` (the
reviewer approves from it alone) and `detail/contract.md` (the implementer and auditor consume it),
resting on the `detail/design.md` that `ideation` already wrote. It reads the shaped design
rather than re-interviewing, keeps each spec to one thin slice, and respects the constitution's house
rules, citing them rather than restating them.

- **Requirements.** Turn each observable behavior into an FR with a success criterion beside it,
  grouped by subsystem or seam.
- **Sequence.** A first-class *When it happens* section covers triggers, ordering, rollout
  conditions, and reversibility.

#### `reviewing-specs`

Point it at a `spec.md`. It reads the spec and the codebase and works its rubric — a decomposition
gate, a six-item self-consistency sweep, and eight review dimensions — and flags any unanswered
template question. It returns BLOCKING / SHOULD_FIX / SUGGESTIONS findings with a verdict.

#### `implementing-specs`

Point it at an approved `spec.md`. It reads the requirements, the contracts, and the success criteria, and
builds one FR-group slice at a time — test-first, watching each success criterion's test fail before making
it pass — verifying each slice before the next rather than coding everything and testing at the end. It builds
on an isolated branch (a worktree where the setup supports one), never the live mainline, and hands the
finishing decision — PR, merge, or discard — back to the human. It keeps a slice ledger so a session
death is recoverable — which slice is done — while the design's *Approach* carries the high-level shape across
the gap, so a resume rebuilds intent from the design rather than `git log`, and the spec never carries HOW. It carries a faked-done anti-patterns catalog and a
root-cause debugging escape hatch for a slice that won't go
green (reproduce first, one hypothesis at a time, and when the contract is the bug, route it to `amend-spec`
rather than reinterpret it). It builds and verifies only — it does not review its own work; the one follow-on
is the independent `auditing-specs` pass, dispatched to a fresh subagent where the harness allows. For the deeper
implementation middle — mature TDD, `systematic-debugging`, worktrees — [`docs/interop.md`](docs/interop.md)
shows how to hand off to superpowers.

#### `auditing-specs`

Point it at an implemented `spec.md`. It traces every requirement to code, runs the spec's own
verification, and reports COMPLIANT or NON_COMPLIANT with `file:line` evidence, tagging each deviation
fix-code or amend-spec. Report-only: it names the next step, and the human takes it.

### The spec

A spec is three documents, one per reader. `spec.md` carries the decision brief: the reviewer
approves or rejects from that page alone. `detail/contract.md` carries the binding sections
(requirements, contracts, verification commands, the worked case): the implementer and auditor load
it whole, with no review-time material diluting their context. `detail/design.md` carries the
reasoning behind the design (current-state facts, failure-mode lenses, impact): a deep reviewer
opens it on doubt, and after approval it is rarely read again.

The full work-item template is the canonical format reference:
[`plugins/spec-monkey/skills/writing-specs/references/spec-template.md`](plugins/spec-monkey/skills/writing-specs/references/spec-template.md).

A large ask decomposes into several work items, each its own spec, linked by frontmatter: `depends_on`
(a hard ship-order) and `relates_to` (a soft sibling link). The cross-cutting rules the work items share
live in the repo constitution (`standards.md` / `CLAUDE.md` / `AGENTS.md`), which stays outside the specs.

### Design principles

- **Discovery lives in the questions.** The interview and risk analysis live in
  `interview-questions.md`; the template only composes the answers.
- **Single source of truth.** Each decision lives in one place and is referenced by ID elsewhere. A shared
  house rule lives in the constitution; a spec cites it, never restates it.
- **Decompose, don't bloat.** A large ask is several work items, not one giant spec. `ideation` splits it
  into one design per decision, linked by `depends_on` / `relates_to`, and each spec stays thin.
- **Handoffs, not auto-run.** Each skill names and offers the next; the human holds the approval gates.
  The human's explicit go-ahead is the gate — the agent may record `approved` for them, but never decides
  it, reads it into silence, or nudges them toward it. Portable, and no hook required.
- **Right-size the ceremony.** The full flow is for work that earns it. `ideation` triages each ask first
  and sends a trivial slice down a fast lane (skip the design, a light contract); the approval gate still
  stands. A one- or two-line change doesn't need a spec at all.
- **Harness-specific glue lives outside the skills.** The skills are plain, portable Markdown that names
  actions, never tools. Anything that needs a hook or a subagent — the optional multi-harness auto-start
  (the env-sniffing bootstrap in [`hooks/`](hooks/)), the fresh-context reviews and audit — is optional and sits
  outside `skills/` or behind a capability check. The per-harness translation from action words to real
  tools lives in [`docs/harness-tools/`](docs/harness-tools/).

### Tooling

[`tools/spec-lint.py`](tools/spec-lint.py) rides the parse contract to settle the mechanical checks — unfilled
placeholders, `FR`↔`SC` pairing, ID uniqueness, status and
`profile` values, and a missing gate record (an `approved` spec with no `approved_by`) — that a script does
faster and cheaper than an LLM reviewer. Stdlib Python, no install, exits nonzero on error so it drops into
CI. `python3 tools/spec-lint.py --status docs/specs` adds a roll-up: every spec's kind, profile, status, and
gate in one table, so work-item state isn't scattered across frontmatter. It judges no engineering; that stays
with `reviewing-specs`. See [`tools/README.md`](tools/README.md).

**Behavioral evidence.** The pressure scenarios in [`tests/`](tests/) ship with a reproducible runner
protocol ([`tests/harness.md`](tests/harness.md)) and genuine recorded runs ([`tests/RESULTS.md`](tests/RESULTS.md)) —
control-vs-skill, real observed output, never asserted or fabricated. The recorded runs show the gate holding
under four escalating "just approve it" nudges and the WHAT/WHEN altitude holding on a HOW-soaked brief, each
across a no-skill control.

**Three version numbers, on purpose.** The package version (`plugin.json` + `marketplace.json`), each skill's
own `version`, and a spec's `spec_monkey:` format version answer three different questions — which release,
which skill revision, which document schema — so they don't move in lockstep. A spec written against an
earlier schema still reads `spec_monkey: "1.4.0"` even at plugin 2.0.0 — it records the schema it was authored
against, not the plugin; the current schema is 1.8.0. The full rule is in
[`AGENTS.md`](AGENTS.md) ("Version — three numbers, three questions").

### Composing with superpowers

spec-monkey owns the front half (ideate, review-design, write, review-spec) and the back half (audit); it composes with
[superpowers](https://github.com/obra/superpowers) for the implementation middle, since a spec-monkey
`contract.md` (WHAT/WHEN) can feed a superpowers plan (HOW) without either system breaking its rules. Who owns
what, and the one seam that matters, is in [`docs/interop.md`](docs/interop.md).

