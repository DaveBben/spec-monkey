# Examples

Worked, end-to-end examples of a spec-monkey spec set, for a made-up project. They exist to show the format
in use — every canonical section filled with real content, one file per reader. They are illustrative, not a
real system.

## `clf-pipeline` — a sentiment classifier training pipeline

One feature — get a HuggingFace dataset onto disk — that `ideation` decomposed into two linked work items:
a full-ceremony loader and a trivial summary line.

```
clf-pipeline/docs/specs/
  dataset-loader/              a FULL work-item spec (kind: work-item) — ideation + writing-specs' output:
    spec.md                    the decision brief, the build contract, the design reasoning.
    detail/contract.md         relates_to: [SPEC-002] (its trivial sibling).
    detail/design.md
  run-summary-log/             a LIGHT work-item spec (profile: light) — the trivial lane:
    spec.md                    one file, reduced section set, depends_on: [SPEC-001], still signed.
```

Read the `dataset-loader` spec first. It defines its own **Dataset** and **SplitManifest** contracts under
*Data & interface contract*, and where a cross-cutting rule binds it — no raw example text in logs — it
names that as a **house rule** (the constitution, `AGENTS.md`) rather than inventing it locally.

Then read `run-summary-log/spec.md` for the contrast: the **trivial lane** (`profile: light`). It is one
`spec.md`, no `detail/` split — one FR/SC, upholding the same no-raw-text-in-logs house rule, still carrying
a full gate record. It is the decomposition made concrete: `depends_on: [SPEC-001]` says it ships after the
loader, and the loader's `relates_to: [SPEC-002]` is the soft sibling link back. The reasoning-and-contract
split is *dropped*, not stubbed with `N/A` — that is what makes the light profile light instead of the full
ceremony hollowed out. Both specs lint clean under the same `tools/spec-lint.py` (the linter checks a light
spec against the reduced schema); run `python3 tools/spec-lint.py examples/clf-pipeline/docs/specs` to see
it, and add `--status` for the roll-up.
