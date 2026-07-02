# Ono Project Inspector

First plugin in the Ono internal AI Marketplace. Inspects an existing repository and gradually builds a structured AI knowledge base for it: `CLAUDE.md`, `AUDIT.md`, a `docs/project/` knowledge base, and per-topic `audits/*.md` files — then syncs approved audit findings back into `CLAUDE.md`.

This is not an implementation assistant. It never modifies source code, never writes feature docs, implementation plans, or source-code patches.

## Commands

| Command | Responsibility |
|---|---|
| `/inspect [repo-path]` | Run or resume the complete guided workflow end to end. Default entry point. |
| `/inspect-status [repo-path]` | Read-only progress report: artifacts present, audit topic counts, enabled/disabled skills, recommended next action. Invokes no skill. |
| `/inspect-topic [topic-name] [repo-path]` | Jump straight to breaking down one audit topic (or the next pending one), skipping the general narrative. |
| `/inspect-approve [repo-path]` | Explicitly approve continuing past the current gate — advance exactly one stage or one topic cycle. |

No command invokes a skill directly by name — every command routes through the `project-inspector` agent, which is the only thing that knows how to sequence skills.

## What it does

1. Runs `project-analysis` to generate `CLAUDE.md` and a concise `AUDIT.md` topic index.
2. Runs `project-docs` to build a descriptive `docs/project/` knowledge base (overview, component inventory, patterns, integrations).
3. Runs `audit-breakdown`, one approved topic at a time, to expand a topic into a Draft audit document under `audits/<topic-slug>/`.
4. Runs `audit-sync` to mark developer-approved topics `Approved` in `AUDIT.md` and fold their HIGH/MEDIUM findings into managed blocks inside `CLAUDE.md`.
5. Stops for developer approval between every stage and between every topic.

See `docs/plugin-workflow.md` for the full step-by-step flow and `docs/architecture.md` for the design rationale.

## Structure

```
.claude-plugin/plugin.json   plugin manifest
agents/project-inspector.md  orchestrator (coordination only, never inspects repos itself; supports 4 invocation modes)
commands/inspect.md          /inspect — full workflow
commands/inspect-status.md   /inspect-status — read-only progress report
commands/inspect-topic.md    /inspect-topic — targeted topic breakdown
commands/inspect-approve.md  /inspect-approve — advance one approval gate
skills/                      vendored inspection skills + registry.json (extensibility seam)
hooks/                       agent-read checkpoint instructions between stages
scripts/                     deterministic verification helpers (slug rules, AUDIT.md consistency)
templates/                   reserved for future skills; unused by current skills by design
docs/                        architecture and workflow documentation
```

## Adding a new inspection skill

1. Place it under `skills/<id>/`.
2. Add one entry to `skills/registry.json`.
3. Optionally add `hooks/after-<id>.md`.

No changes to the manifest, agent, command, or existing hooks are required. See `docs/architecture.md` for details.

## Status

All four workflow stages are implemented and enabled:

- `project-analysis` — enabled (stage 1)
- `project-docs` — enabled (stage 2)
- `audit-breakdown` — enabled (stage 3)
- `audit-sync` — enabled (stage 4)
