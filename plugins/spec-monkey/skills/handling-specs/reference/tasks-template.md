<!--
TASKS TEMPLATE: the sibling file `tasks.md`, written by the spec-decomposer AFTER the spec is
approved. It is the HOW: one readable section per task, each self-contained enough for an AI
implementor to build without re-reading the whole spec. It stays empty through `draft` and
`reviewed`; the spec-decomposer fills it at `decomposed`.

Division of labour:
  spec.md   owns WHAT / WHY and the intent IDs (FR-NNN, SC-NNN). It carries NO file manifest.
  tasks.md  owns the HOW: the file manifest, exact symbols, and run commands, PER TASK. The
            decomposer derives these from the spec's design plus the real codebase.

Each task cites the FR-/SC- ids it implements/verifies, so work traces back to intent. Ordering:
`depends_on` is the source of truth; `wave` is derived from it (the parallel batch). Keep tasks
readable top-to-bottom: one section each, NOT one dense wide table.
-->

> **Spec:** [`spec.md`](spec.md)

# Tasks

## T1 — <single-concern summary>
`wave 1` · `~120 LOC` · implements FR-001 · verifies SC-002 · depends_on —

**Files**
| path | mode | symbol |
|------|------|--------|
| <path/to/file> | new \| modify \| delete \| context | <function / class / const the task touches> |

**Run**
```
<exact command(s) that gate this task — e.g. uv run pytest tests/test_x.py -q ; uv run ruff check src>
```

**Notes** *(optional: only the non-obvious)*
- <pattern to mirror · ordering trap · gotcha the implementer would otherwise hit>

- [ ] done

## T2 — <single-concern summary>
`wave 2` · `~300 LOC` · implements FR-003, FR-004 · depends_on T1

**Files**
| path | mode | symbol |
|------|------|--------|
| <path/to/file> | modify | <symbol> |

**Run**
```
<command(s)>
```

- [ ] done

<!--
Column / field notes:
- mode: new (file must not exist yet) | modify | delete | context (read-only, informs the task).
- The trace line is `wave N · ~EST LOC · implements FR-… · verifies SC-… · depends_on …`.
  Keep the `~` on the estimate. `verifies SC-…` only on tasks that carry the test for a criterion.
- `T#*` (asterisk) marks an optional task, e.g. extra hardening tests.
- A file belongs to ONE parallel task. If two tasks edit the same file, sequence them via depends_on.
-->
