# Hook: after-project-docs

Type: agent-read checkpoint instruction.
Consumed by: `agents/project-inspector.md`, immediately after the `project-docs` skill reports completion.

Not a Claude Code `hooks.json` event — a checklist the agent follows itself.

## Checklist

1. Confirm the skill's completion report lists all four files as written: `docs/project/overview.md`, `docs/project/components.md`, `docs/project/patterns.md`, and `docs/project/integrations.md`. If any is missing, report the gap to the developer rather than advancing.
2. Confirm the report states that `CLAUDE.md`, `AUDIT.md`, and `audits/` were **not** modified. `project-docs` is descriptive-only and must not touch those files or the audit index. If its output contract was violated, flag it and do not attempt to fix it yourself.
3. `project-docs` requires `CLAUDE.md` to already exist. If the skill reported that it stopped because `CLAUDE.md` was missing, do not treat the stage as complete — report that `project-analysis` must run first.
4. This stage always has `requiresApproval: true` in the registry — stop here regardless of how clean the output looks. Do not auto-invoke `audit-breakdown`.
5. In your stage-transition report, remind the developer that `project-docs` (a descriptive inventory) and `audit-breakdown` (evaluative findings) are complementary — the next stage produces per-topic audit findings, not a repeat of the docs.

## Why this exists

`project-docs` sits between `project-analysis` and `audit-breakdown` and consumes the former's output (`CLAUDE.md`) without knowing about the latter at runtime. This hook is where the orchestrator confirms the descriptive knowledge base was produced within its read-only contract before the workflow moves on to evaluative audit work.
