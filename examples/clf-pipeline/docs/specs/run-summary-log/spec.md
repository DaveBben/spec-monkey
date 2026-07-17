---
spec_monkey: "1.8.0"
id: SPEC-002
kind: work-item
profile: light              # trivial lane: one file, the droppable sections dropped (not "N/A"). See spec-template.md "The light profile".
title: Log a run summary line when the loader finishes
status: approved
approved_by: [@dave]
approved_date: 2026-07-15
created: 2026-07-15
updated: 2026-07-15
owners: [@dave]
standards: AGENTS.md
depends_on: [SPEC-001]
relates_to: []
supersedes: []
---

# Spec: Log a run summary line when the loader finishes

<!--
This is a LIGHT-PROFILE (trivial-lane) spec: a single spec.md, no detail/ split. One obvious
approach, no live failure modes — a reviewer reads it in a glance. It still gets signed. This is
the second half of one feature: the dataset-loader (SPEC-001) decomposed into a full-ceremony
loader and this trivial summary line, so it declares depends_on: [SPEC-001]. The full
design/contract split (detail/*.md) is dropped here, not stubbed with "N/A". Contrast with the
full-ceremony dataset-loader spec.
-->

## Goal

When a loader run finishes, an operator sees one summary line — dropped-row count and per-split
sizes — instead of scrolling the log to piece it together.

## The request

> Print a one-line summary at the end of a loader run so we can eyeball what it produced.

## Requirements & success criteria

### Run summary

**Behavior**: after the loader has written all splits, it emits one INFO log line stating how many
rows it dropped and how many examples landed in each of train, val, and test.

**Requirements & success criteria**

- **FR-001**: WHEN a loader run completes successfully, the system SHALL log exactly one summary
  line reporting the dropped-row count and the per-split example counts (train, val, test).
  - **SC-001**: after a run over the 11-row worked fixture, the completion log contains one line
    reading `run complete: dropped=1 train=8 val=1 test=1`, and no example `text` appears anywhere
    in that line.

## Constraints & non-functional bounds

**Constraints**: the summary line upholds the repo's no-raw-example-`text`-in-logs house rule (the
constitution, `AGENTS.md`). Counts and split names only; never a row's contents. No new data
contract: the numbers come from the SplitManifest the loader (SPEC-001) already writes.

## Verification approach & commands

**Artifacts to author.** Must exist in the diff; proves every SC below.
- [ ] Author one test: run the loader over the worked fixture, assert the summary line's exact text, and
      assert no fixture `text` value appears in the captured log. Proves SC-001 and the no-raw-text-in-logs bound. Runs here.

**Gates to pass.**
- Runs here: `uv run pytest tests/test_loader.py -k summary_line` — covers SC-001 and the no-raw-text-in-logs bound.

**Worked case**

- **Given:** the 11-row fixture from SPEC-001 (10 valid, 1 dropped) with `seed=42`. · **When:** the
  loader completes. · **Then:** the final log line reads `run complete: dropped=1 train=8 val=1 test=1`
  and contains none of the fixture's example text.
