<!--
TASKS TEMPLATE — the sibling file `tasks.md` that holds a spec's Tasks table.
Lives at `docs/specs/{slug}/tasks.md`, next to `spec.md`. Split out of the spec (progressive
disclosure) so a large table never bloats the core. The `spec.md` `Tasks` section is a pointer
to this file. Referenced ids (F#, #N) live in `spec.md`; join by id across the two files.
-->

> **Spec:** [`spec.md`](spec.md)

# Tasks

<!-- Tasks are the HOW; they stay empty through `draft` and `reviewed` so the spec is
     approved on intent. The spec-decomposer writes the table. `depends_on` is the source
     of truth for ordering; `wave` is derived from it (the parallel batch). `est_diff` is
     an estimated diff-line count (keep the `~`). `id*` marks optional work (e.g. extra
     tests). Each task cites the acceptance criteria (`#N`) it satisfies and the
     Files / Change Manifest rows (`F#`) it touches — both live in `spec.md`; the executor
     flips `done`. -->
| id | task | files | AC | depends_on | wave | est_diff | done |
|----|------|-------|----|------------|------|----------|------|
| T1 | <single-concern summary> | F1, F2 | #1, #2 | — | 1 | ~120 | [ ] |
| T2 | <…> | F3 | #3 | T1 | 2 | ~300 | [ ] |
