---
name: audit-sync
description: >-
  Marks developer-approved audit topics as Approved in AUDIT.md and syncs
  their HIGH and MEDIUM findings into managed blocks inside CLAUDE.md
  (Caution Areas and Important Files), keeping CLAUDE.md a living document.
  Only runs on explicit developer approval of specific topics — it never
  decides approval itself. Re-runs replace the managed blocks, never
  duplicate them. Part of the project-inspector workflow.
---

# Audit Sync Skill

## Purpose

Close the loop of the project-inspector workflow: once the developer approves a Draft audit, its key findings should be visible to every future Claude Code session via `CLAUDE.md` — without anyone re-reading the full audit file.

This skill is step 4 of the workflow:

1. `project-analysis` creates `CLAUDE.md` and `AUDIT.md`.
2. `project-docs` creates the `docs/project/` knowledge base.
3. `audit-breakdown` creates one Draft audit per approved run.
4. `audit-sync` (this skill) marks developer-approved topics as `Approved` and syncs findings into `CLAUDE.md`.

## Output Contract

This skill may modify only:

```text
<repository-root>/AUDIT.md      — Status column of explicitly approved rows only
<repository-root>/CLAUDE.md     — content INSIDE the audit-sync managed blocks only
```

It never touches source code, `audits/` files, or `docs/`.

## The Approval Rule

**Only the developer approves audits.** This skill acts only on topics the developer explicitly names as approved in this conversation, for example:

```text
The networking audit is approved, sync it.
Approve topics 2 and 5.
/inspect-sync networking
```

Never infer approval from a topic being old, complete-looking, or previously discussed. If the developer says "sync audits" without naming topics, list the Draft topics and ask which ones are approved.

## Managed Blocks in CLAUDE.md

This skill writes only between these marker pairs (created by the project-analysis template):

```markdown
<!-- audit-sync:caution-areas:start -->
...replaced on every sync...
<!-- audit-sync:caution-areas:end -->

<!-- audit-sync:important-files:start -->
...replaced on every sync...
<!-- audit-sync:important-files:end -->
```

Rules:

- On every run, regenerate the full content of each block from ALL currently-Approved topics in `AUDIT.md` — not just the newly approved ones. This makes re-runs idempotent: no duplication, no stale entries for topics that were re-drafted.
- If the markers are missing from `CLAUDE.md` (e.g., an older or hand-written file), add the marker pairs at the end of the `## Caution Areas` and `## Important Files` sections respectively. If those sections don't exist either, ask the developer before adding them.
- Never modify anything outside the markers.

## Core Workflow

1. Ask for the repository root (or confirm the current one if already established in the conversation).
2. Read `AUDIT.md`; list topics with `Status = Draft` and `Status = Approved`.
3. Confirm with the developer exactly which Draft topics are approved in this run.
4. Show a confirmation summary and wait for explicit approval to proceed.
5. For each newly approved topic: update its `AUDIT.md` row `Status` from `Draft` to `Approved`. Touch nothing else in the row.
6. Read every `Approved` topic's audit file (path from the `File` column).
7. Extract findings tagged `[HIGH]` and `[MEDIUM]` (the format written by `audit-breakdown`).
8. Regenerate both managed blocks in `CLAUDE.md`.
9. Report completion.

## Step 4 Confirmation Summary

```text
Here's what I'm about to do:

- Repository: <path>
- Newly approving: <topic list>
- Already approved (will be re-synced): <topic list or none>

I will update:
  ✓ AUDIT.md — Status → Approved for: <topics>
  ✓ CLAUDE.md — regenerate the two audit-sync managed blocks

I will not touch:
  ✗ anything in CLAUDE.md outside the managed blocks
  ✗ audits/ files
  ✗ source code

Shall I proceed?
```

## Block Content Format

### Caution Areas block

One entry per finding, grouped by topic, most severe first:

```markdown
<!-- audit-sync:caution-areas:start -->
_From approved audits (managed by audit-sync — do not edit by hand):_

**Networking** ([audit](audits/networking/networking-audit.md))
- 🔴 HIGH — `APIClient` retry logic can duplicate POST requests (`src/api/client.ts`)
- 🟡 MEDIUM — Auth token refresh is not thread-safe (`src/api/auth.ts`)

**Managers and Singletons** ([audit](audits/managers-and-singletons/managers-and-singletons-audit.md))
- 🔴 HIGH — `SessionManager` holds mutable global state accessed from 14 files
<!-- audit-sync:caution-areas:end -->
```

Keep each finding to one line: severity, symbol/area, short consequence, file path. The full detail stays in the audit file — this block is a pointer layer, not a copy.

### Important Files block

```markdown
<!-- audit-sync:important-files:start -->
| Approved audit | File |
|----------------|------|
| Networking | audits/networking/networking-audit.md |
| Managers and Singletons | audits/managers-and-singletons/managers-and-singletons-audit.md |
<!-- audit-sync:important-files:end -->
```

## Completion Report

```text
Audit sync complete.

Approved in this run:
  ✓ <topics>

CLAUDE.md updated:
  ✓ Caution Areas block — <n> findings from <m> approved audits
  ✓ Important Files block — <m> audit pointers

AUDIT.md updated:
  ✓ <topics> → Approved

Every future Claude Code session in this repo now sees these findings via CLAUDE.md.
```

## Hard Constraints

- Never mark a topic Approved unless the developer explicitly named it as approved in this conversation.
- Never write outside the two managed blocks in `CLAUDE.md`.
- Never modify `audits/` files, `docs/`, or source code.
- Never sync findings below MEDIUM severity (LOW stays in the audit file only).
- Never copy full finding bodies into `CLAUDE.md` — one line per finding plus a link.
- Never reproduce secret values, even if an audit file quotes variable names.
- If an approved topic's audit file is missing or unreadable, skip it, keep its status as Draft, and report the problem — do not guess its findings.
