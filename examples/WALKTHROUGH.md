# Walkthrough: Adding User Avatars to a Sample Express API

This walkthrough narrates a complete `/feature` → `/execute` cycle against
a hypothetical Express + PostgreSQL API. It shows what happens at each
phase, what artifacts are produced, and where the user intervenes.

## The Setup

**Project**: A REST API with users, posts, and comments. Express +
TypeScript, PostgreSQL via Drizzle ORM, Jest for testing. The repo has
a CLAUDE.md and spec.md from a previous `/onboard` run.

**Feature request**: "Add user avatar support — users should be able to
upload a profile picture that displays on their posts."

---

## Phase 0: Deep Research

### Step 0: Clarify Intent

The user runs:
```
/feature add user avatar support — upload profile picture, display on posts
```

The skill confirms:
> Before I research the codebase, let me confirm I understand:
> - **What**: Add avatar upload to user profiles, display avatars on posts
> - **Why**: Users want visual identity in the app
>
> Is that right?

User: "Yes, but we want to store avatars in S3, not the database."

The skill then asks about external context and domain risks. The user
mentions: "We use the `@company/upload-lib` package for all S3 uploads —
don't implement S3 directly."

### Steps 1-4: Research

The skill reads CLAUDE.md, spec.md, explores the codebase, and writes
`.claude/features/add-user-avatar/research.md`:

```markdown
# Research: Add user avatar support

## Feature Intent
**What**: Avatar upload for user profiles, displayed on posts
**Why**: Visual identity for users

## Codebase Findings
- User model at `src/models/user.ts:15` — no avatar field currently
- Post response serializer at `src/serializers/post.ts:8` — includes
  `author: { id, name }` — needs avatar URL added
- Existing upload pattern at `src/services/document-upload.ts:22` uses
  `@company/upload-lib` for S3 — MUST follow this pattern
- Test pattern at `tests/services/document-upload.test.ts:1` — uses
  fake S3 client from `@company/upload-lib/testing`
...
```

### Step 5: Devil's Advocate

The devil's advocate agent returns challenges:

```markdown
## Devil's Advocate Challenge

### Assumptions
1. `[BLOCKING]` **Avatar images are small enough for synchronous upload**:
   If users upload 10MB photos, the upload endpoint will timeout. The
   existing document upload at `src/services/document-upload.ts:45` has
   a 5MB limit — see `MAX_FILE_SIZE` constant. — see `src/config.ts:12`
   > Response:

### Edge Cases
2. `[ADVISORY]` **User deletes account with avatar**: The S3 object
   remains orphaned. `src/services/user.ts:89` `deleteUser()` doesn't
   clean up external resources. — see `src/services/user.ts:89`
   > Response:
```

### Step 6: User Responds

User reviews and responds:
- Item 1: "Good catch — we'll use the same 5MB limit. Acknowledged."
- Item 2: "Out of scope — we'll handle orphan cleanup separately."

---

## Phase 1: Impact Map

```markdown
# Impact Map: Add user avatar support

## Files Affected

| File | Action | Symbols | Reason |
|------|--------|---------|--------|
| `src/models/user.ts` | modify | `users` table | Add avatarUrl column |
| `src/services/avatar.ts` | create | `uploadAvatar()` | New upload service |
| `src/routes/users.ts` | modify | `router` | Add PUT /users/:id/avatar |
| `src/serializers/post.ts` | modify | `serializePost()` | Include avatar URL |
| `drizzle/0005_add_avatar.sql` | create | — | Migration |
| `tests/services/avatar.test.ts` | create | — | Avatar service tests |
| `tests/routes/users-avatar.test.ts` | create | — | Endpoint tests |
```

The user confirms and the skill continues.

---

## Phase 2: Plan

The skill writes `.claude/features/add-user-avatar/plan.md` with actual
code snippets:

```markdown
# Plan: Add user avatar support

**Date**: 2026-04-12
**Status**: Draft
<!-- annotation-cycle: 0/6 -->

## Assumptions
- **avatarUrl is a nullable text column** — users without avatars return null
- **@company/upload-lib handles S3 directly** — no raw AWS SDK calls
- **5MB file size limit** — matching existing document upload pattern

## Implementation

### 1. Database Migration

**Files**: `drizzle/0005_add_avatar.sql`, `src/models/user.ts`
**Proposed code**:
```sql
ALTER TABLE users ADD COLUMN avatar_url TEXT;
```
...
```

### Step 3: Plan Verification

The plan-verifier agent checks all references and confirms:
- `src/models/user.ts:15` exists and matches
- `@company/upload-lib` is in package.json
- `MAX_FILE_SIZE` exists at `src/config.ts:12`
- No discrepancies found

### Step 4: User Reviews

The user sees the plan and adds one annotation:
```
@FIX: use drizzle:generate for the migration, not raw SQL
```

After processing, the skill updates the plan and presents:
> "I've addressed your annotation (cycle 1/6). The migration now uses
> drizzle:generate. Review or say 'approve'."

User: "approve"

---

## Phase 4: Task Breakdown

The skill produces 5 tasks:

```markdown
# Tasks: Add user avatar support

## 1. Database

### 1.1 Add avatar column to users table
- **Files**: `src/models/user.ts`, `drizzle/0005_add_avatar.sql`
- **Symbol**: `users` table schema
- **Reference**: See `drizzle/0004_add_bio.sql` for migration pattern
- **MUST**: Add nullable `avatar_url` text column using drizzle:generate
- **Do NOT**: modify any existing columns or other tables
- **Done when**: Migration runs, column exists, existing tests pass
```

Each task also gets a JSON file in `.claude/features/add-user-avatar/tasks/`.

The skill presents:
> "Task breakdown complete — 5 tasks. When ready, run:
>
> `/execute .claude/features/add-user-avatar/`"

---

## /execute

The user runs `/execute .claude/features/add-user-avatar/`.

### Pre-flight

- Branch: creates `feature/add-user-avatar`
- Task JSON validation: all 5 JSONs valid, paths exist, commands runnable
- Environment check: `pg_isready` passes
- Test baseline: 142 tests passing, 0 known failures
- spec.md freshness: last updated 12 days ago — OK

### Task Execution (Task 1 of 5)

**Implement**: code-implementor reads task_0.json, adds the `avatar_url`
column to the schema, runs `drizzle:generate`, and verifies tests pass.

**Commit**: `"Task 0: Add avatar column to users table"`

**Light-review**: No findings. Clean.

**Execution state**: `execution-state.json` updated, task_0 marked DONE.

### Tasks 2-5

Similar cycle. By task 3, the dispatch count reaches 16. After task 4
(dispatch count 20), the context gate fires:

> "Context checkpoint — 4 tasks completed (20 agent dispatches), 1 remaining.
> Check your context usage. Continue or start a fresh session?"

User checks: context at 35%. Says "continue."

Task 5 completes. All 5 tasks DONE.

### Post-Implementation

- Full test suite: 149 tests passing (7 new), 0 failures
- Deep review: 4 agents dispatched. Findings:
  - SHOULD_FIX: Missing input validation on file type (avatar.ts:15)
  - SUGGESTION: Add cache headers to avatar URL responses
- Fix SHOULD_FIX, commit "Deep review fixes"
- Second deep review: clean. APPROVE.
- spec.md Current State updated: "Users can upload profile avatars (S3)"
- PR created: `feature/add-user-avatar` → `main`

### Result

```
## Execution Summary

Branch: feature/add-user-avatar
PR: #47 — https://github.com/example/api/pull/47
Tasks: 5 completed, 0 failed
Commits: 7 (5 tasks + 1 review fix + 1 spec update)

### Task Results
- Task 0: Add avatar column — DONE
- Task 1: Avatar upload service — DONE
- Task 2: Upload endpoint — DONE
- Task 3: Post serializer avatar — DONE
- Task 4: Edge case tests — DONE

### Review
- /light-review: 0 issues
- /deep-review: 1 SHOULD_FIX fixed, 1 SUGGESTION noted

### Next Steps
Ready for review.
```

---

## Key Takeaways

1. **Research before planning** — the devil's advocate caught the file
   size assumption before any code was written
2. **The user intervened once** — one `@FIX:` annotation on the migration
   approach. Everything else was automated.
3. **Domain knowledge was captured** — the `@company/upload-lib` constraint
   came from the user in Phase 0 and was enforced throughout
4. **5 tasks, ~45 minutes total** — from feature description to PR
5. **The context gate fired but didn't force a restart** — context was
   healthy enough to finish in one session
