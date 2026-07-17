---
spec_monkey: "1.8.0"
id: SPEC-001
kind: work-item
title: Ingest a HuggingFace dataset into leak-free versioned splits
status: draft
approved_by: []
approved_date:
created: 2026-07-14
updated: 2026-07-14
owners: [@dave]
standards: AGENTS.md
depends_on: []
relates_to: [SPEC-002]
supersedes: []
---

# Spec: Ingest a HuggingFace dataset into leak-free versioned splits

## Decision brief

### Goal

A named HuggingFace dataset becomes a versioned, on-disk Dataset with train/val/test splits that share no
example, so the trainer and evaluator have a reproducible input.

### Decisions to sign off

- **Split strategy** · owner: ml-lead: stratified 80/10/10 by label, deterministic by seed, over a
  simpler random split. Random splitting can skew label balance on smaller datasets and does not guarantee
  leak-free splits on its own. *Is stratified 80/10/10 the right default for this project?*

### Blocking open questions

- **Open (blocking):** the canonical HuggingFace `dataset_id` and `source_revision` are not yet pinned;
  gates reproducibility and the first Dataset version.

### Blast radius & reversibility

- **Blast radius:** a bad split corrupts every model and metric trained on that Dataset version, invisibly.
- **Reversibility:** fully reversible until a model trains on a version — versions are additive and
  regenerable; deleting an unused version is safe.

### Rollout / cutover gate

The first Dataset version must pass the leak check (SC-004) — and the balance bound where the dataset has at
least 1,000 rows — before the trainer work item may start against it.

## Contents

**How to read this spec:** the design — the ask, the drivers, the approach — was reviewed and approved at the
design gate; the approver signs the contract off from the brief above, opening the files below on doubt. The
implementer and the auditor load the contract whole. A deep reviewer reads everything.

- [The contract](detail/contract.md) — what the implementer builds and the auditor checks:
  *Requirements & success criteria* · *Constraints & non-functional bounds* · *Data & interface contract* ·
  *When it happens* · *Out of scope* · *Known limitations & honest gaps* (incl. coverage exceptions) ·
  *Verification approach & commands* (incl. the worked case)
- [The design](detail/design.md) — the approved reasoning:
  *The request* · *Drivers* · *What's true today* · *Approach* · *Failure modes* (incl. contingencies) ·
  *Who & what this touches* · *Open questions & assumptions*
