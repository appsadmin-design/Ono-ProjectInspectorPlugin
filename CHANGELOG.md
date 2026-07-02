# Changelog

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
