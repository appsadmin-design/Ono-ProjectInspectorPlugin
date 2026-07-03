# Ono Project Inspector

First plugin in the Ono internal AI Marketplace. Inspects an existing repository and gradually builds a structured AI knowledge base for it: `CLAUDE.md`, `AUDIT.md`, a `docs/project/` knowledge base, and per-topic `audits/*.md` files — then syncs approved audit findings back into `CLAUDE.md`.

This is not an implementation assistant. It never modifies source code, never writes feature docs, implementation plans, or source-code patches.

## Commands

| Command | Responsibility |
|---|---|
| `/inspect [repo-path]` | Run or resume the complete guided workflow end to end. Default entry point. |
| `/inspect-status [repo-path]` | Read-only progress report: artifacts present, audit topic counts, enabled/disabled skills, recommended next action. Invokes no skill. |
| `/inspect-topic [topic-name] [repo-path]` | Jump straight to breaking down one audit topic (or the next pending one), skipping the general narrative. |
| `/inspect-approve [repo-path]` | Approve the current gate: finalize the reviewed audit Draft (`Draft` → `Approved`) and break down the next topic, or advance one non-repeatable stage. |
| `/inspect-sync [repo-path]` | Run the `audit-sync` documentation-maintenance tool on demand: refresh the managed blocks in `CLAUDE.md` from approved audits, verify consistency, repair drift. Not part of the linear workflow. |

No command invokes a skill directly by name — every command routes through the `project-inspector` agent, which is the only thing that knows how to sequence skills.

## What it does

1. Runs `project-analysis` to generate `CLAUDE.md` and a concise `AUDIT.md` topic index.
2. Runs `project-docs` to build a descriptive `docs/project/` knowledge base (overview, component inventory, patterns, integrations).
3. Runs the **breakdown → approve loop** over audit topics, one at a time:
   - `audit-breakdown` expands one topic into a `Draft` audit document under `audits/<topic-slug>/` and stops for review.
   - `audit-approve` finalizes the reviewed Draft (`Draft` → `Approved` in `AUDIT.md`) — it is the single owner of that transition — after which the next topic is broken down automatically.
4. Stops for developer approval between every stage and after every Draft.
5. Separately, on demand, `audit-sync` (documentation maintenance) folds the HIGH/MEDIUM findings of approved topics into managed blocks inside `CLAUDE.md` and checks the index for drift. It never approves anything and is not part of the linear workflow.

See `docs/plugin-workflow.md` for the full step-by-step flow and `docs/architecture.md` for the design rationale.

## Structure

```
.claude-plugin/plugin.json   plugin manifest
agents/project-inspector.md  orchestrator (coordination only, never inspects repos itself; supports 5 invocation modes)
commands/inspect.md          /inspect — full workflow
commands/inspect-status.md   /inspect-status — read-only progress report
commands/inspect-topic.md    /inspect-topic — targeted topic breakdown
commands/inspect-approve.md  /inspect-approve — finalize the reviewed Draft, then continue
commands/inspect-sync.md     /inspect-sync — on-demand documentation-sync maintenance
skills/                      vendored skills + registry.json (extensibility seam); includes internal inspection-state
hooks/                       agent-read checkpoint instructions between stages
scripts/                     deterministic helpers (slug rules, AUDIT.md consistency, .ono/state.json state)
templates/                   reserved for future skills; unused by current skills by design
docs/                        architecture and workflow documentation
```

The plugin also maintains a committed, portable state file at `<target-repo>/.ono/state.json` (owned by the internal `inspection-state` skill) so an interrupted inspection resumes exactly where it left off. `AUDIT.md` remains the source of truth; the state file only mirrors it.

## Adding a new inspection skill

1. Place it under `skills/<id>/` and list it in `plugin.json`'s `skills[]`.
2. Add one entry to `skills/registry.json` (with `type` = `workflow` or `internal`, and `role`/`pairsWith`/`workflowRole` as appropriate).
3. Optionally add `hooks/after-<id>.md` and a command.

A skill that fits an existing shape (a linear stage, a breakdown-approve loop partner, or an on-demand maintenance tool) needs no change to the agent or existing hooks; only a genuinely new orchestration pattern does. See `docs/architecture.md` for details.

## Status

All skills are implemented and enabled:

- `inspection-state` — enabled (internal infrastructure, auto-invoked; not user-facing)
- `project-analysis` — enabled (stage 1, inspection)
- `project-docs` — enabled (stage 2, inspection)
- `audit-breakdown` — enabled (stage 3, inspection — breakdown half of the loop)
- `audit-approve` — enabled (stage 3, inspection — approval half of the loop)
- `audit-sync` — enabled (maintenance tool, outside the linear workflow)
