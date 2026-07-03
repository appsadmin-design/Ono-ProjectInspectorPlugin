# Architecture

## Design goal

The plugin must let new inspection skills (e.g. `ui-ux-inspector`, `architecture-inspector`, `security-inspector`, `performance-inspector`) be added later without editing the agent, the command, or existing hooks. The mechanism for that is a **skill registry**, not a hardcoded workflow.

## Components

```
commands/inspect.md          thin entry point, starts the agent
agents/project-inspector.md  orchestrator: reads the registry, sequences skills, enforces approval gates
skills/registry.json         the extensibility seam: what skills exist, order, dependencies, hooks
skills/<id>/SKILL.md         the actual analysis skills (vendored, unmodified logic)
hooks/*.md                   checkpoint instructions the agent reads before/after each stage
scripts/*.ts                 deterministic (non-LLM) helpers hooks can invoke to verify skill output
```

## Why a registry instead of hardcoded orchestration

If `agents/project-inspector.md` contained explicit steps like "run project-analysis, then run audit-breakdown," adding `security-inspector` later would require editing that file — and every other skill's ordering assumptions would need re-checking too. Instead, the agent's logic is generic:

```
read registry -> consider only workflowRole: inspection entries -> sort by stage ->
  for each entry:
    check requires[] exist -> run before-hook (if any) ->
    invoke skill by id -> run after-hook (if any) ->
    stop if requiresApproval
  paired entries (role + pairsWith, e.g. audit-breakdown <-> audit-approve)
    iterate as one loop: break down -> review gate -> approve -> continue
```

Entries with `workflowRole: maintenance` (e.g. `audit-sync`) are skipped by this loop entirely — they run only on demand in `maintenance` mode. Adding a skill that fits one of the existing shapes (a linear stage, a loop partner, or a maintenance tool) is a data change (one JSON entry, one skill folder, optionally one hook file and one command); introducing a genuinely new orchestration shape is the rare case that also updates the agent.

## Approval model

The inspection skills each enforce their own "ask, confirm, wait for approval" steps internally. The registry's `requiresApproval` flag is a second, orchestration-level gate: even if a skill technically could keep going, the agent will not chain into the next stage without the developer explicitly saying so.

Stage 3 makes the approval model explicit by giving finalization its own skill. `audit-breakdown` creates a `Draft` and stops (`requiresApproval: true` — the review gate); `audit-approve` is the **single owner** of the `Draft` -> `Approved` transition and runs only on explicit developer approval (`requiresApproval: false`, because invoking it *is* the approval). After a clean approval the agent immediately breaks down the next topic, so the developer drives the loop one reviewed topic at a time. This keeps each skill single-responsibility: breakdown drafts, approve finalizes, and `audit-sync` (maintenance) only ever reflects already-`Approved` topics into `CLAUDE.md` — it never approves and never writes `AUDIT.md`.

## Hooks are agent-read instructions, not Claude Code hook events

`hooks/*.md` are plain markdown checklists the orchestrating agent reads at defined points in its own loop (`before-inspect`, `after-<skill-id>`). They are not wired into a `hooks.json` / `PreToolUse` / `PostToolUse` mechanism. This was a deliberate simplification: it keeps the extensibility model uniform (everything the agent consults — skills, hooks — is markdown it reads and follows), and avoids requiring shell-level hook plumbing for what is fundamentally an LLM-driven review-and-approve workflow. If a future skill needs a hard, code-enforced gate (e.g. actually blocking a write), that would be a separate, explicit addition — not assumed here.

## Deterministic verification scripts

Skills produce markdown by LLM judgment, which is good for prose but not ideal for exact-match bookkeeping like slugs and table cell values. `scripts/slugify.ts` and `scripts/update-audit-index.ts` give the `after-audit-breakdown` and `after-audit-approve` hooks a way to double-check the skill's own edit against a fixed rule — verifying the row's status and that the permanent `File` reference matches the deterministic slug on both `Draft` and `Approved` rows — without the agent or a script ever being allowed to rewrite the file itself. Discrepancies are reported to the developer, not silently patched.

## Extending the plugin

To add a new skill, e.g. `security-inspector`:

1. Place the skill under `skills/security-inspector/` and list it in `plugin.json`'s `skills[]` array.
2. Add one entry to `skills/registry.json` with its `stage`, `role`/`pairsWith`/`workflowRole`, `requires`, `produces`, and `requiresApproval`.
3. Optionally add `hooks/after-security-inspector.md` if the stage needs a checkpoint, and a command under `commands/` (also listed in `plugin.json`) if it needs a dedicated entry point.

If the new skill fits an existing orchestration shape — a linear inspection stage, a breakdown-approve-style loop partner, or an on-demand maintenance tool — no change to `agents/project-inspector.md` or existing hooks is needed; the agent's generic logic already handles it via the registry fields. Only a genuinely new orchestration pattern (as the breakdown-approve loop itself was) requires updating the agent.
