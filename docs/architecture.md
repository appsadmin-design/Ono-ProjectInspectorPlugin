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
read registry -> sort by stage -> for each enabled entry:
  check requires[] exist -> run before-hook (if any) ->
  invoke skill by id -> run after-hook (if any) ->
  stop if requiresApproval
```

Adding a skill is a data change (one JSON entry, one skill folder, optionally one hook file), not a code change.

## Approval model

All four skills (`project-analysis`, `project-docs`, `audit-breakdown`, `audit-sync`) already enforce their own "ask, confirm, wait for approval" steps internally. The registry's `requiresApproval` flag is a second, orchestration-level gate: even if a skill technically could keep going, the agent will not chain into the next stage without the developer explicitly saying so. This mirrors `audit-breakdown`'s own one-topic-per-approval rule, generalized to the whole workflow.

## Hooks are agent-read instructions, not Claude Code hook events

`hooks/*.md` are plain markdown checklists the orchestrating agent reads at defined points in its own loop (`before-inspect`, `after-<skill-id>`). They are not wired into a `hooks.json` / `PreToolUse` / `PostToolUse` mechanism. This was a deliberate simplification: it keeps the extensibility model uniform (everything the agent consults — skills, hooks — is markdown it reads and follows), and avoids requiring shell-level hook plumbing for what is fundamentally an LLM-driven review-and-approve workflow. If a future skill needs a hard, code-enforced gate (e.g. actually blocking a write), that would be a separate, explicit addition — not assumed here.

## Deterministic verification scripts

Skills produce markdown by LLM judgment, which is good for prose but not ideal for exact-match bookkeeping like slugs and table cell values. `scripts/slugify.ts` and `scripts/update-audit-index.ts` give the `after-audit-breakdown` hook a way to double-check the skill's own edit against a fixed rule, without the agent or a script ever being allowed to rewrite the file itself — discrepancies are reported to the developer, not silently patched.

## Extending the plugin

To add a new skill, e.g. `security-inspector`:

1. Place the skill under `skills/security-inspector/`.
2. Add one entry to `skills/registry.json` with its `stage`, `requires`, `produces`, and `requiresApproval`.
3. Optionally add `hooks/after-security-inspector.md` if the stage needs a checkpoint.

No changes to `plugin.json`, `agents/project-inspector.md`, `commands/inspect.md`, or any existing hook file are needed.
