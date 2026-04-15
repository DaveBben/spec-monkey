# Project Spec Template

Copy-paste starting point for spec.md. Fill in each section following the guidance in
`spec-section-guidance.md`. Sections marked (optional) can be omitted if not applicable.

**For multi-domain projects** (2+ subsystems): this root spec stays slim and system-level.
Domain-specific content (external deps, testing gaps, known issues, tech debt, domain
boundaries) moves to `{domain-dir}/spec.md`. See `domain-spec-template.md`.

## Contents

- [What This Project Does](#what-this-project-does)
- [Why This Project Exists](#why-this-project-exists)
- [Current State](#current-state) — most important
- [Architecture Overview](#architecture-overview) — includes shared external deps
- [Testing Strategy](#testing-strategy)
- [Deployment & Infrastructure](#deployment--infrastructure)
- [Boundaries & Constraints](#boundaries--constraints)
- [Ownership](#ownership)
- [Domain Specs](#domain-specs) — multi-domain only
- [Known Issues](#known-issues)
- [Tech Debt](#tech-debt)

---

```markdown
# [Project Name] — Project Spec

**Last updated**: [YYYY-MM-DD]
**Last verified**: [YYYY-MM-DD]
**Status**: [Draft | Active | Needs Update] <!-- See spec-section-guidance.md for status definitions -->

> This spec represents current state, not aspirational state. Update it whenever
> implementation changes. The "Current State" section is the most important — keep it accurate.

---

## Table of Contents

- [What This Project Does](#what-this-project-does)
- [Why This Project Exists](#why-this-project-exists)
- [Current State](#current-state)
- [Architecture Overview](#architecture-overview)
- [Testing Strategy](#testing-strategy)
- [Deployment & Infrastructure](#deployment--infrastructure)
- [Boundaries & Constraints](#boundaries--constraints)
- [Ownership](#ownership)
- [Known Issues](#known-issues)
- [Tech Debt](#tech-debt)

---

## What This Project Does

[2-4 sentences. What the project is, who uses it, and why it exists. No implementation details.
This grounds every decision — if an AI is unsure about something, this section tells it what
the project is trying to accomplish.]

---

## Why This Project Exists

[The problem this project solves and what motivated its creation. If this project is a
micro-service or component of a larger system, describe that context here — what system does
it serve and what role does it play?]

- [e.g., "Manual invoice processing was costing the ops team 20 hours/week and producing errors"]
- [e.g., "Part of the billing platform — this service owns payment intent creation; the
  notifications service handles emails downstream"]

---

## Current State

<!-- THIS IS THE MOST IMPORTANT SECTION. Write 2-4 concrete sentences describing what is
actually implemented and working today. No future tense.
For multi-domain projects: keep this system-level. Each domain spec has its own Current State. -->

[e.g., "The API accepts invoice submission via POST /invoices and stores them in PostgreSQL.
Reconciliation runs nightly via a cron job and creates Linear tickets for discrepancies.
The Stripe integration is complete; NetSuite integration is stubbed with a TODO in
src/integrations/netsuite.ts and currently throws NotImplementedError."]

---

## Architecture Overview

[High-level description of the major components and how they relate. An ASCII diagram is
fine for complex systems. Focus on inter-domain data flows, not domain internals.]

```
[Client] → [API Gateway] → [Service Layer] → [Database]
                                ↓
                          [Event Bus] → [Workers]
```

**Key components:**
- [Component and what it does, e.g., "src/api/ — Express routes, one file per resource"]
- [Component and what it does]

### Shared External Dependencies

[Dependencies used by 2+ domains or the project as a whole. Domain-specific deps
go in that domain's spec.md instead.]

| Dependency | Normal Behavior | Failure Behavior | Constraints |
|------------|----------------|------------------|-------------|
| [e.g., PostgreSQL] | [connection pool, max 20] | [circuit breaker trips after 5 failures, returns 503] | [query timeout: 10s] |

**Degraded mode:** [What the system does when a shared dependency is unavailable.]

### Architecture Decisions (optional)

| Decision | Rationale | Date | Alternatives Considered |
|----------|-----------|------|------------------------|
| [e.g., "PostgreSQL over MongoDB"] | [e.g., "Relational data model, strong consistency needed for billing"] | [YYYY-MM-DD] | [e.g., "MongoDB — rejected due to transaction complexity"] |

---

## Testing Strategy

[Project-wide testing infrastructure only. Domain-specific coverage gaps and
conventions go in domain specs.]

**Framework:** [name and version]
**Location:** [where test files live relative to source]
**Naming:** [convention, e.g., "*.test.ts co-located with source files"]

**Test types:**
- **Unit tests**: [how to run them]
- **Integration tests**: [how to run them, any setup needed]
- **E2E tests** (optional): [framework, how to run]

**Testing conventions:**
- [e.g., "Use real database for integration tests, not mocks"]

---

## Deployment & Infrastructure

[How the app gets from code to production.]

**CI/CD:** [e.g., "GitHub Actions — tests run on every PR, deploy to staging on merge to
`staging`, deploy to production on manual approval after merge to `main`"]

**Environments:**
- **Production**: [URL, hosting platform, e.g., "AWS ECS behind ALB"]
- **Staging**: [URL, how it's triggered]

**Infrastructure:** [Key infra components, e.g., "RDS PostgreSQL, ElastiCache Redis,
S3 for file storage, CloudFront CDN"]

**Environment variables:** [How they're managed, e.g., "AWS Secrets Manager — see
.env.example for the full list of required variables"]

---

## Boundaries & Constraints

[Project-wide boundaries only. Domain-specific boundaries go in domain specs.]

### Always Do
- [e.g., "Run tests before committing"]
- [e.g., "Update the Current State section in spec.md after significant implementation changes"]

### Ask First
- [e.g., "Before changing database schema or migrations"]
- [e.g., "Before modifying CI/CD configuration"]

### Never Do
- [e.g., "Never commit .env files or secrets"]
- [e.g., "Never modify files in vendor/ or generated/"]

---

## Ownership

- **Team/Owner**: [team name or individual]
- **Contact**: [Slack channel, email, or on-call rotation]

---

## Domain Specs

[For multi-domain projects only. Remove this section for single-domain projects.]

| Domain | Path | Owns |
|--------|------|------|
| [e.g., Billing] | `src/billing/spec.md` | [e.g., "payment intents, refunds, Stripe integration"] |
| [e.g., Auth] | `src/auth/spec.md` | [e.g., "authentication, sessions, JWT handling"] |

Each domain spec contains: domain-specific current state, conventions,
interface contracts, external deps, testing gaps, boundaries, known issues,
and gotchas. Loaded via `.claude/rules/` when working in that directory.

---

## Known Issues

[Cross-cutting issues only. Domain-specific issues go in domain specs.
Write "None known." if clean.]

- **[Issue]**: [description, severity, workaround if any]

---

## Tech Debt

[Cross-cutting tech debt only. Domain-specific debt goes in domain specs.]

- [e.g., "No database connection pooling — using a single connection. Acceptable at current
  load (<100 req/min) but will need addressing before scaling. Ticket: #87."]
```

> **Target: 60-100 lines for multi-domain projects.** The root spec is a navigation
> layer, not a knowledge dump. Domain-specific content lives in domain specs.
> For single-domain projects, include all content in the root spec (no domain
> specs needed) — target 100-200 lines.
