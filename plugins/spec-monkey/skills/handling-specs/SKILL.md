---
name: handling-specs
description: "Canonical spec format reference. Invoke before writing, reviewing, or decomposing a spec so you work against the right template, section schema, and parse contract. Informational — it defines the format; it performs no action."
---

# Handling Specs: the canonical format

Invoke this skill whenever you write, review, or decompose a spec. It is the single source
of truth for the spec's shape.

## Where specs live

One folder per spec: `docs/specs/{slug}/`. `{slug}` is a short kebab-case name for the
change. The folder holds two files:

- **`spec.md`** is the reviewer-facing spec: frontmatter, a verifiable Goal line, then the
  numbered sections. It is DESIGN, covering what/why, requirements, how-it-fits, data
  contracts, edge cases, blast radius, verification, rollout. The durable contract; always loaded.
- **`tasks.md`** is the per-task implementation breakdown: file manifest, exact symbols, and run
  commands, one readable section per task. It stays empty until the spec is approved, then the
  spec-decomposer fills it.

The spec's audit trail is **git**, not a file. Run `git log -- docs/specs/{slug}/` to see the
history of every change to the spec.

## Altitude: what goes where

The spec is written directly by `create-spec` (there is no separate plan file). It stays at
**design altitude**: everything a human signs off on, and no more. The moment a line names
*which* file or symbol to edit, it belongs in `tasks.md`, not the spec.

- **Stays in the spec:** requirements (FR-NNN), data contracts, edge cases, success criteria
  (SC-NNN), verification strategy, rollout. **Data contracts stay** even though they look
  implementation-ish, because types are contracts a reviewer signs.
- **Moves to `tasks.md`:** the file / change manifest, the exact symbols, the run commands, and
  task ordering. The decomposer derives these from the spec's design plus the real codebase,
  after approval.

## Progressive disclosure

`spec.md` is the always-loaded core; a reader reviewing intent loads only it. `tasks.md` is
loaded on demand by an executor. It holds the implementation HOW and is populated *late*,
exactly when a reader wants intent rather than the breakdown. In `spec.md`, the `Tasks` section
stays in the numbered body as a one-line **pointer** to `tasks.md`. Keeping the manifest and
commands out of the spec holds it at a stable, readable size regardless of task count.

The **Activity Log** section is likewise a pointer, but its target is **git history**, not a
file. Don't hand-maintain a change log in the spec. Git already records who changed what, when.

## The template

The full template, frontmatter plus the eleven sections, is in
[`reference/spec-template.md`](reference/spec-template.md). The Tasks sibling file has its own
template: [`reference/tasks-template.md`](reference/tasks-template.md). Read them before you
write or review.

The format is **`schema_version: 3`**.

A spec is **human-first Markdown** with a few machine-readable islands. The `.md` is the source
of truth. A CLI parses it *to* JSON; nobody hand-authors JSON directly.

## Single source of truth

Each decision lives in EXACTLY ONE section; every other place that needs it references it by ID
(`FR-001`, `SC-001`, a config name) instead of re-stating it. Restating a decision in a second
section is the top cause of spec bloat and drift. Concretely: Edge Cases and Verification cite
the FR they exercise rather than re-prosing it; a Data-contract type is signed once and
referenced; a rationale is stated once and referenced. If two sections say the same thing, one
is a copy. Cut it to a reference.

## Parse contract (lightweight: what a CLI / linter relies on)

1. **Frontmatter**: YAML between the leading `---` fences. Holds every queryable scalar
   (`schema_version`, `id`, `status`, `depends_on`, `supersedes`). Never bury one in prose.
2. **Sections**: split the body on `^## (\d+)\. <name>`. The canonical section set and order
   are the schema; treat them as frozen. (Sub-structure splits on `**Group**` labels.)
3. **IDs are the join keys**: `FR-NNN` (requirements) and `SC-NNN` (success criteria), unique
   within the spec. `tasks.md` cites them (`implements FR-001 · verifies SC-002`) to build the
   task → requirement / success-criterion trace. There is **no** EARS `#N` and **no** `F#`
   manifest id in the spec. The file manifest lives in `tasks.md`, which owns its own `T#` ids.
4. **Light prose conventions**: `- **Key:** value` → a key/value field; `**Group**` on its own
   line → introduces the list/table/block that follows; a fenced block (Data contracts) →
   captured with its language tag. Everything else is opaque prose keyed by its heading.
5. **`tasks.md`**: one section per task (`## T#`), each with a `**Files**` table and a `**Run**`
   fenced block. A parser reads it as an extension of the spec, joining on `FR-/SC-` ids.

## When you cite a section

Always reference a section by its **header name**, never its number. Numbering can shift;
headers don't.
