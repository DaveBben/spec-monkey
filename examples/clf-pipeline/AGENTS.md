# AGENTS.md — clf-pipeline constitution

The house rules every work item in this project must respect. A spec cites a rule here
rather than restating it; the implementer and the audit hold the build to them.

## House rules

- **No raw example text in logs or metadata.** No raw example `text` value is ever written
  to a log line, a metric, or artifact metadata. Only counts, ids, and split names may be
  emitted. This keeps dataset content out of observability, where it would leak.
