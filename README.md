# Think-Plan-Execute (TPE)

A pipeline that attempts to ground its design choices in empirical research, keeping humans in architectural decisions while AI handles investigation and implementation.

## Installation

Install as a Claude Code plugin:

```bash
# Add the marketplace
/plugin marketplace add <owner>/backlog-driven-development

# Install the plugin
/plugin install tpe@tpe-marketplace
```

Or test locally:

```bash
claude --plugin-dir ./backlog-driven-development
```

All skills are namespaced under `tpe:` (e.g., `/tpe:think`, `/tpe:execute`).

---

## Routing

```
Is this a new codebase or one you haven't onboarded yet (brownfield)?
  └── Yes → /tpe:onboard  (creates CLAUDE.md + spec.md)

Can the change be understood by a single diff?
  └── Yes → Use vanilla Claude Code with plan mode. No skills needed.

Is it a bug?
  └── Yes → /tpe:bug → then /tpe:execute

Is it a new feature or capability?
  └── Yes → /tpe:think → /tpe:plan → /tpe:execute

Don't know where the change lives or what it touches?
  └── Yes → /tpe:think (investigation phase surfaces this before any planning)

Want a second set of eyes on what you're writing?
  └── Yes → /tpe:review (four-agent parallel review — never writes code)
```

---

## Skills

| Skill | What it does | Artifacts produced |
|---|---|---|
| `/tpe:onboard` | Sets up context for a new or existing codebase | `CLAUDE.md`, `spec.md` |
| `/tpe:think` | Investigates scope, surfaces consequences, gets human decisions before planning | `brainstorm.md` |
| `/tpe:plan` | Reads approved brainstorm, writes implementation plan, decomposes to task JSONs | `plan.md`, `plan.json`, `task_{N}.json` |
| `/tpe:execute` | Implements tasks: dispatches code-implementor per task, regression checks, deep review | Branch, commits |
| `/tpe:bug` | Captures symptom, investigates root cause, produces task JSONs directly | `task_{N}.json` (no plan.md intermediate) |
| `/tpe:review` | Four-agent parallel review (security, reliability, maintainability, performance) | Consolidated review report |

---

## Skill Details

### `/tpe:onboard`

Run this first on any codebase — new or existing. The skill reads the repository, asks targeted questions about purpose, tech stack, conventions, and constraints, then produces:

- **`CLAUDE.md`** — quick-reference context that Claude Code loads on every conversation
- **`spec.md`** — living specification describing what the system does, key decisions, and known constraints

Once onboarded, `/tpe:think` and `/tpe:bug` have rich context to work from, increasing the chances that AI agents implement changes without breaking existing data contracts.

### `/tpe:think`

Investigative conversation that clarifies scope, constraints, and approach — **before** any planning or implementation. The human makes architectural decisions here; the AI investigates and surfaces consequences.

Runs three parallel Explore agents (blast radius, existing patterns, data shapes), synthesizes findings, and asks:
1. Impact surface — what does this change touch?
2. At-risk tests — which tests must not break? (requires human confirmation)
3. Approach — human states their approach first, AI presents alternatives, human decides

Produces **`brainstorm.md`** — the contract between think and plan. Plan will not start until Status is `Approved`.

### `/tpe:plan`

Reads the approved `brainstorm.md`, deepens targeted investigation, and produces:

- **`plan.md`** — implementation plan with real code, constraints, and pattern references (100–200 lines)
- **`plan.json`** — machine-readable plan metadata
- **`task_{N}.json`** files — one per vertical slice, each with: files (max 4), symbol, single reference, dependency chain, at-risk tests, acceptance criteria (GIVEN/WHEN/THEN), verification command, and explicit scope boundaries

Runs a `plan-verifier` agent to fact-check all file:line references before producing task JSONs.

### `/tpe:execute`

Takes a plan directory and processes all task JSONs:

1. Pre-flight: branch safety, validate task JSONs, test baseline, spec freshness
2. For each task: dispatch `code-implementor` agent at appropriate complexity tier (haiku/sonnet/opus, maxTurns 20–75)
3. Trust-but-verify: run `regressionCheck` after each task — hard stop on failure
4. Per-task commit after verification passes (rollback granularity)
5. Handoff check when next task depends on the completed one
6. `/tpe:review` after all tasks for the repo
7. Full test suite + lint

Does not push or create PRs

### `/tpe:bug`

Bugs start from a symptom, not a change request, so they skip the think→plan path. Guides through: symptom description, reproduction steps, root cause investigation via two parallel agents, then produces task JSONs directly:

- `task_0`: write the failing reproduction test
- `task_1`: fix (blocked by task_0, done when repro test goes green)
- `task_2`: edge case tests (optional)

Same task JSON schema as features — `/tpe:execute` does not need to know whether it's running a bug or feature.

If the investigation finds that the bug touches many files across multiple concerns, the skill warns and offers to escalate to `/tpe:think` → `/tpe:plan` → `/tpe:execute`.

### `/tpe:review`

Four agents run in parallel, each focused on one dimension:

| Agent | Focus |
|---|---|
| `security-reviewer` | Injection, access control, data exposure |
| `reliability-reviewer` | Correctness, race conditions, resource lifecycle |
| `maintainability-reviewer` | Readability, compatibility, conventions |
| `performance-reviewer` | N+1 queries, blocking I/O, resource leaks |

Results are consolidated into a single report. Used automatically by `/tpe:execute` after all tasks for a repo complete. Can also be invoked standalone on any code.

---

## Agents

| Agent | Role | Used by |
|---|---|---|
| `plan-verifier` | Fact-checks plan.md references against the codebase | `/tpe:plan` |
| `code-implementor` | Reads task JSON, implements code, verifies against acceptance criteria | `/tpe:execute` |
| `task-handoff-checker` | Checks export/import consistency between completed and dependent tasks | `/tpe:execute` |
| `security-reviewer` | Injection, access control, data exposure | `/tpe:review` |
| `reliability-reviewer` | Correctness, race conditions, resource lifecycle | `/tpe:review` |
| `maintainability-reviewer` | Readability, compatibility, conventions | `/tpe:review` |
| `performance-reviewer` | N+1 queries, blocking I/O, resource leaks | `/tpe:review` |

---

## Artifact Storage

Plans and tasks are stored per-project inside `.claude/`:

```
{project-root}/
  .claude/
    features/
      {slug}/
        brainstorm.md
        plan.md
        plan.json
        tasks/
          task_0.json
          task_1.json
    bugs/
      {slug}/
        tasks/
          task_0.json
          task_1.json
        repro-test.[ext]
```

`brainstorm.md` records human decisions and confirmed at-risk tests. `plan.md` contains the implementation approach and constraints. `task_{N}.json` files contain individual tasks: files, symbol, dependency chain, at-risk tests, acceptance criteria, and verification commands.

---

## Design Principles

**Human decides before AI plans.** `/tpe:think` surfaces consequences and alternatives; the human chooses the approach. The AI never recommends — it presents options and waits.

**Brainstorm → Plan → Execute is a verified handoff chain.** Each skill consumes artifacts from the previous one. Plan blocks if brainstorm is unapproved or has unverified tests. Execute validates task JSONs before touching code.

**Context density over context volume.** Every field in a task JSON must earn its place. Wrong context is worse than no context — so at-risk tests are human-confirmed, references are capped at one, and files are capped at four.

**Regression prevention is the primary quality metric.** Targeted at-risk tests (human-confirmed, specific) reduce regressions 72%. Generic TDD instructions without targeted context increase them. The pipeline is designed around this finding.

**Skip the skills when you don't need them.** One- or two-line changes do not need a plan file. Use vanilla Claude Code with plan mode.

---

## Research Foundations

Design choices map to empirical findings on how coding agents fail and how humans lose skill when AI removes friction. 

### Agent effectiveness

| Principle | Key finding | Pipeline response |
|-----------|------------|-------------------|
| **Context precision over volume** | Wrong context is worse than none ([2602.08316](https://arxiv.org/abs/2602.08316)). Context types interfere ([2503.20589](https://arxiv.org/abs/2503.20589)). Structured context degrades review across all models tested ([2603.26130](https://arxiv.org/abs/2603.26130)). | Artifacts capped (brainstorm 200 lines, plan 200, task JSONs per-task only). Every field must earn its place. |
| **Task decomposition** | Highest-ceiling intervention: +10–40pp ([2510.07772](https://arxiv.org/abs/2510.07772), [2311.05772](https://arxiv.org/abs/2311.05772)). Agents collapse from 74% to 11% on feature-level tasks ([2602.10975](https://arxiv.org/abs/2602.10975)). | `/tpe:plan` decomposes into vertical slices, hard max 4 files per task. |
| **Targeted test context** | Test dependency info reduced regressions 72%; generic TDD instructions *increased* them 42% ([2603.17973](https://arxiv.org/abs/2603.17973)). | Human-confirmed at-risk tests, multi-agent triangulated. Never instructs "write tests first." |
| **Single agent** | Matches or beats multi-agent at fraction of cost ([2604.02460](https://arxiv.org/abs/2604.02460), [2505.18286](https://arxiv.org/abs/2505.18286)). Weaker model + strong scaffolding wins ([2512.10398](https://arxiv.org/abs/2512.10398)). | One agent per task. Haiku/Sonnet implements, Opus orchestrates. |
| **Instruction brevity** | Shorter instructions quadrupled resolution ([2603.17973](https://arxiv.org/abs/2603.17973)). Context length alone degrades performance even with perfect retrieval ([2510.05381](https://arxiv.org/abs/2510.05381)). | Task JSONs carry intent, not procedures. |
| **Agent overconfidence** | Agents predict 77% success at 22% actual ([2602.06948](https://arxiv.org/abs/2602.06948)). | Trust-but-verify: orchestrator runs `regressionCheck`, never trusts agent self-report. |
| **Instruction fade-out** | System instructions lose influence as context fills ([2603.05344](https://arxiv.org/abs/2603.05344)). | Re-surfaces constraints before each verification; pauses every 6 tasks. |
| **Quality erosion** | Code quality eroded in 80% of trajectories ([2603.24755](https://arxiv.org/abs/2603.24755)). Security vulnerabilities increased 37.6% after 5 iterations ([2506.11022](https://arxiv.org/abs/2506.11022)). | Per-task commits, handoff checks, size/drift warnings, human pause gates. |

### Human cognition

| Principle | Key finding | Pipeline response |
|-----------|------------|-------------------|
| **Predict-before-reveal** | Predicting before seeing results improves learning and defends against anchoring ([2410.08922](https://arxiv.org/abs/2410.08922), [SSRN:6097646](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=6097646)). | `/tpe:think` asks human to predict at-risk tests and propose approach before revealing findings. |
| **Human decides, AI investigates** | Formulating hypotheses produces deeper learning than following suggestions ([2505.08063](https://arxiv.org/abs/2505.08063)). | AI never recommends. Human chooses approach, confirms boundaries, signs off scope. |
| **Batch reveals** | Most AI interactions stay in a single metacognitive phase, skipping planning and evaluation ([2511.04144](https://arxiv.org/abs/2511.04144)). | Three sequential batches (impact → tests → approach), each gated on human response. |
| **Inquiry over delegation** | Conceptual inquiry builds skill; code delegation erodes it ([2601.20245](https://arxiv.org/abs/2601.20245), [2506.08872](https://arxiv.org/abs/2506.08872)). | `/tpe:think` = inquiry mode (human). `/tpe:execute` = delegation (agent). |

### Cross-cutting

- **Verified context only.** Multi-agent convergence for at-risk tests, grep-verified dependency chains, orchestrator-run regression checks. No stage trusts its inputs blindly ([2602.08316](https://arxiv.org/abs/2602.08316)).
- **Progressive compression.** brainstorm (~200 lines) → plan (~200 lines) → task JSONs (~50–80 lines). Semantic density over token count ([2604.07502](https://arxiv.org/abs/2604.07502)).
- **Human gates at decision points, automation at verification points.** Human chooses approach, approves plan, handles push/PR. Regression checks, reference verification, and handoff validation are mechanical.
