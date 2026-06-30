---
name: handling-specs
description: "Canonical spec format reference. Invoke before writing, reviewing, or decomposing a spec so you work against the right template, section schema, and parse contract. Informational — it defines the format; it performs no action."
---

# Handling Specs — the canonical format

Invoke this skill whenever you write, review, or decompose a spec. It is the single source
of truth for the spec's shape.

## Where specs live

One file per spec: `docs/specs/{slug}/spec.md`. `{slug}` is a short kebab-case name for the
change.

## The template

The full template — frontmatter plus the fourteen sections — is in
[`reference/spec-template.md`](reference/spec-template.md). Read it before you write or
review.

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
   Assumptions, Files / Change Manifest, Tasks); fenced blocks (` ```lang `) → captured with
   the language tag (Data Model & Contracts, Verification); marker regions
   `<!-- AC:BEGIN -->` … `<!-- AC:END -->` → criteria, each `#N <text>`.
4. **Two prose conventions** — `- **Key:** value` → a key/value field; `**Group**` on its own
   line → introduces the list/table/block that follows. Everything else is opaque prose keyed
   by its heading.
5. **IDs are the join keys** — `FR-NNN`, `NFR-NNN`, `SC-NNN`, AC `#N`, file `F#`, task `T#` —
   unique within a spec. The traceability graph (task → AC → FR → file) is built from these.

## When you cite a section

Always reference a section by its **header name**, never its number — numbering can shift,
headers don't.
