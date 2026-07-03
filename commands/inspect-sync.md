---
description: Run the Ono Project Inspector documentation-maintenance tool (audit-sync) on demand — sync CLAUDE.md with approved audits, verify AUDIT.md consistency, and repair drift.
argument-hint: "[repository-path]"
---

Invoke the `project-inspector` agent in **`maintenance` mode** (see the agent's "Invocation Modes" section in `agents/project-inspector.md`).

This runs the `audit-sync` skill, which is a maintenance tool and **not** part of the normal inspection workflow (its `registry.json` entry has `workflowRole: maintenance`). Use it once one or more topics are `Approved` — or any time later — to:

- regenerate the `audit-sync` managed blocks inside `CLAUDE.md` from all currently-Approved topics,
- verify `AUDIT.md` consistency,
- detect documentation drift, and
- repair broken audit references.

It does not mark anything Approved — finalizing a Draft is `audit-approve`'s job (via `/inspect-approve`). If an argument was provided, treat it as the repository path in case conversation context was lost; otherwise rely on context from the current session. The underlying skill still asks for confirmation before writing.

$ARGUMENTS
