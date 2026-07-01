---
name: handling-specs
description: "Canonical spec format reference. Invoke before writing, reviewing, or decomposing a spec so you work against the right template, section schema, and parse contract. Informational — it defines the format; it performs no action."
---

# Handling Specs — the canonical format

Invoke this skill whenever you write, review, or decompose a spec. It is the single source
of truth for the spec's shape.

## Where specs live

One folder per spec: `docs/specs/{slug}/`. `{slug}` is a short kebab-case name for the
change. The folder holds two files:

- **`spec.md`** — the core spec: frontmatter plus the fourteen sections. The durable
  contract; always loaded.
- **`tasks.md`** — the Tasks table. Empty until the spec is approved, then the
  spec-decomposer fills it.

The spec's audit trail is **git**, not a file — `git log -- docs/specs/{slug}/` is the
history of every change to the spec.

## Progressive disclosure

`spec.md` is the always-loaded core. The **Tasks** table lives in a sibling file, `tasks.md`,
loaded on demand — so the noise it accrues never bloats the core. A reader reviewing intent
loads only `spec.md`; an executor loads `tasks.md`. In `spec.md`, the `Tasks` section stays in
the numbered body as a one-line **pointer** to `tasks.md` — the section still exists, its
content just lives one hop away.

The **Activity Log** section is likewise a pointer, but its target is **git history**, not a
file: `git log` on the spec folder is the audit trail. Don't hand-maintain a change log in the
spec — git already records who changed what, when.

Why split Tasks out: the table can grow to dozens of rows after decomposition, and it's
populated *late* — exactly when a reader wants intent, not the implementation breakdown.
Keeping it out holds `spec.md` at a stable, readable size regardless of task count.

## The template

The full template — frontmatter plus the fourteen sections — is in
[`reference/spec-template.md`](reference/spec-template.md). The Tasks sibling file has its own
template: [`reference/tasks-template.md`](reference/tasks-template.md). Read them before you
write or review.

The format is **`schema_version: 2`**.

A spec is **structured Markdown**: a human-first document with machine-readable islands. The
`.md` is the source of truth; a CLI parses it *to* JSON — no one hand-authors JSON.

## Single source of truth

Each decision lives in EXACTLY ONE section; every other place that needs it references it by ID
(FR-001, NFR-001, SC-001, F3, a config name) instead of re-stating it. Restating a decision in a second
section is the top cause of spec bloat and drift. Concretely: Edge Cases and Verification cite the
FR they exercise rather than re-prosing it; the Glossary never redefines a type that the Data Model
& Contracts section gives a
signature; a `resolved` Clarifications row points to its FR/config home, it doesn't repeat the
answer; a rationale is stated once (Assumptions) and referenced. If two sections say the same
thing, one is a copy — cut it to a reference.

## Parse contract (what a CLI / linter relies on)

1. **Frontmatter** — YAML between the leading `---` fences. Holds every queryable scalar
   (`schema_version`, `status`, `depends_on`, `supersedes`). Never bury a
   queryable value in prose.
2. **Sections** — split the body on `^## (\d+)\. <name>`. The canonical section set and order
   are the schema; treat them as frozen. (Sub-structure splits on `^### `.)
3. **Typed islands** — Markdown tables → row-objects keyed by column (Clarifications,
   Assumptions, Files / Change Manifest, and the Tasks table); fenced blocks (` ```lang `) →
   captured with the language tag (Data Model & Contracts, Verification); marker regions
   `<!-- AC:BEGIN -->` … `<!-- AC:END -->` → criteria, each `#N <text>`. The `Tasks` section
   in `spec.md` is a pointer: the table lives in `tasks.md`, and a parser follows the pointer
   and reads that sibling file as an extension of the spec (same `T#` join keys). The
   `Activity Log` section is a pointer to **git history** — there is no log island to parse.
4. **Two prose conventions** — `- **Key:** value` → a key/value field; `**Group**` on its own
   line → introduces the list/table/block that follows. Everything else is opaque prose keyed
   by its heading.
5. **IDs are the join keys** — `FR-NNN`, `NFR-NNN`, `SC-NNN`, AC `#N`, file `F#`, task `T#` —
   unique within a spec. The traceability graph (task → AC → FR → file) is built from these.

## When you cite a section

Always reference a section by its **header name**, never its number — numbering can shift,
headers don't.
