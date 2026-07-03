# Changelog

## 0.4.0 — 2026-07-03

Redesigned the audit approval workflow around a dedicated `audit-approve` skill, so each skill has a single responsibility. Previously, "approve & continue" only advanced to the next Draft and never finalized the reviewed topic — Drafts accumulated with none marked `Approved`. Approval is now its own step.

- Added `audit-approve` (stage 3, approval half of the new breakdown → approve loop). It is the single owner of the `Draft` → `Approved` transition: it validates the topic is `Draft`, flips its `AUDIT.md` row to `Approved`, confirms the permanent file reference, and never generates a topic or touches `CLAUDE.md`.
- Reworked `agents/project-inspector.md`: Stage 3 is now a breakdown → review → approve → continue loop. After a clean `audit-approve`, the agent immediately breaks down the next `Pending Breakdown` topic and stops at its review gate; when all topics are `Approved` it reports Stage 3 complete. Added a fifth invocation mode (`maintenance`) and redefined `resume` to finalize the reviewed Draft.
- `/inspect-approve` now finalizes the reviewed Draft (runs `audit-approve`) and continues the loop, instead of merely advancing one breakdown cycle.
- Added `/inspect-sync` and a `maintenance` mode to run `audit-sync` on demand.
- Repurposed `audit-sync` as a documentation-maintenance tool only (`workflowRole: maintenance`), removed from the linear workflow. It no longer marks anything `Approved` and now treats `AUDIT.md` as read-only — it syncs approved findings into the `CLAUDE.md` managed blocks, verifies consistency, detects drift, and repairs references inside its blocks.
- Extended `skills/registry.json` with `role`, `pairsWith`, and `workflowRole` fields to express the breakdown/approve pairing and the maintenance role. `audit-breakdown` unchanged in responsibility (create one Draft, stop) — only its completion/approval wording now points at `audit-approve`.
- Added hook `after-audit-approve`; updated `after-audit-sync` (maintenance, no approval) and `after-audit-breakdown` (points at approve). Extended `scripts/update-audit-index.ts` to verify the file reference on `Approved` rows too.
- Updated `.claude-plugin/plugin.json` (registered the new skill and command, bumped to 0.4.0), `README.md`, `docs/plugin-workflow.md`, and `docs/architecture.md`.
- No changes to any target repository's source code.

## 0.3.0 — 2026-07-02

Completed the four-stage workflow and prepared for distribution.

- Vendored and enabled `audit-sync` (stage 4) — marks developer-approved topics `Approved` in `AUDIT.md` and regenerates the two managed blocks in `CLAUDE.md` from all Approved topics. Previously registered as a disabled placeholder.
- Vendored and enabled `project-docs` (stage 2) — builds a descriptive `docs/project/` knowledge base (overview, component inventory, patterns, integrations). Adapted to this plugin's conventions: read-only shell rules instead of the external guard-marker/PreToolUse mechanism, direct read-only inspection instead of a `repo-scanner` agent, and all four templates embedded in `SKILL.md` instead of external template files.
- Fixed a cross-skill coupling gap: `project-analysis`'s `CLAUDE.md` template now emits the `audit-sync:important-files` and `audit-sync:caution-areas` managed-block markers that `audit-sync` depends on, with placeholder content and a rule to preserve them.
- Renumbered the workflow to four stages: `project-analysis` → `project-docs` → `audit-breakdown` → `audit-sync`, all enabled in `skills/registry.json`.
- Added hook checkpoints `after-project-docs` and `after-audit-sync`.
- Updated `.claude-plugin/plugin.json` skills list and bumped the version.
- Added `.claude-plugin/marketplace.json` so the plugin is installable via the Ono internal AI Marketplace, plus a `.gitignore` for repository hygiene.
- Updated `README.md` and `docs/plugin-workflow.md` to document all four enabled stages.
- No changes to any target repository's source code.

## 0.2.0 — 2026-07-02

Expanded command API.

- Added `/inspect-status` — read-only progress report, invokes no skill.
- Added `/inspect-topic` — targeted single-topic breakdown, bypassing the general narrative.
- Added `/inspect-approve` — explicit, discoverable approval to advance one gate.
- Added an "Invocation Modes" section to `agents/project-inspector.md` (`full`, `status`, `targeted`, `resume`) so the same orchestrator dispatches correctly per command, with no duplicated orchestration logic.
- Updated `.claude-plugin/plugin.json` commands list.
- Updated `README.md` and `docs/plugin-workflow.md` to document all four commands.
- No changes to skill business logic (`project-analysis`, `audit-breakdown`) and no changes to any target repository's source code.

## 0.1.0 — 2026-07-02

Initial release.

- Added orchestrating agent `agents/project-inspector.md`.
- Added `/inspect` command.
- Vendored existing skills: `project-analysis`, `audit-breakdown`.
- Added `skills/registry.json` as the extensibility seam for future skills.
- Registered `audit-sync` as a disabled placeholder — skill not yet implemented locally.
- Added agent-read hook checkpoints: `before-inspect`, `after-project-analysis`, `after-audit-breakdown`.
- Added deterministic verification scripts: `slugify.ts`, `update-audit-index.ts`.
- Added architecture and workflow documentation.
