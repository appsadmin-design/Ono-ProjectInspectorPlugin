# Plugin Workflow

## Commands

The plugin exposes five commands, all thin wrappers around `agents/project-inspector.md`, which dispatches on the mode each command declares (see the agent's "Invocation Modes" section):

- `/inspect [repo-path]` — `full` mode. Runs the guided end-to-end inspection workflow described below.
- `/inspect-status [repo-path]` — `status` mode. Read-only snapshot of progress; invokes no skill.
- `/inspect-topic [topic-name] [repo-path]` — `targeted` mode. Runs one `audit-breakdown` cycle directly for the named (or next pending) topic.
- `/inspect-approve [repo-path]` — `resume` mode. Approves the current gate: finalizes the reviewed audit Draft (runs `audit-approve`) and continues the breakdown-approve loop, or advances one non-repeatable stage.
- `/inspect-sync [repo-path]` — `maintenance` mode. Runs the `audit-sync` documentation-maintenance tool on demand (not part of the linear workflow).

A developer never needs to know a skill's name to use the plugin — every command already knows which skill(s) it needs and lets the agent handle sequencing.

## Running the guided workflow

```text
/inspect /path/to/repository
```

or just `/inspect`, and the agent will ask for the repository path.

## Step-by-step

1. **Startup** — the agent reads `skills/registry.json` and `hooks/before-inspect.md`. Via the internal `inspection-state` skill it detects whether the repo was already inspected (reading `<repo>/.ono/state.json`), handles any plugin/schema version mismatch or migration, and computes a resume pointer, then reports the planned stage order (and any disabled or maintenance-only skills).
2. **Stage 1 — project-analysis** — the agent checks no prerequisites are needed, invokes `project-analysis`, which asks its own intake questions, confirms, then writes `CLAUDE.md` and `AUDIT.md` only. The agent runs `hooks/after-project-analysis.md`, reports the topic count, and stops for approval.
3. **Developer reviews** `CLAUDE.md` and `AUDIT.md`, then approves continuing.
4. **Stage 2 — project-docs** — the agent confirms `CLAUDE.md` exists, invokes `project-docs`, which reads the existing context, inspects source read-only, and writes the four `docs/project/*.md` files (overview, components, patterns, integrations). It never touches `CLAUDE.md`, `AUDIT.md`, or `audits/`. The agent runs `hooks/after-project-docs.md` and stops for approval.
5. **Developer reviews** the `docs/project/` files, then approves continuing.
6. **Stage 3 — the breakdown-approve loop.** This stage iterates over `Pending Breakdown` topics, one at a time, using two paired skills:
   1. **Break down (audit-breakdown)** — the agent invokes `audit-breakdown`, which asks which `Pending Breakdown` topic to process (default: first), confirms, then writes exactly one `audits/<slug>/<slug>-audit.md` and sets that row to `Draft`. The agent runs `hooks/after-audit-breakdown.md` (deterministic slug/status check) and **stops at the Draft's review gate**.
   2. **Developer reviews** the Draft audit file.
   3. **Approve (audit-approve)** — on approval (`/inspect-approve` or "approved, continue"), the agent invokes `audit-approve`, which validates the topic is `Draft`, flips its `AUDIT.md` row to `Approved`, and confirms the permanent file reference. The agent runs `hooks/after-audit-approve.md`. `audit-approve` has no approval gate of its own — running it *is* the approval.
   4. **Continue** — immediately after a clean approval, the agent breaks down the next `Pending Breakdown` topic (back to 6.1) and stops at its review gate. No second Draft is ever generated without a review in between.
   5. **Stage 3 complete** — when no `Pending Breakdown` topics remain and no `Draft` is open (every topic is `Approved`), the agent reports Stage 3 complete and offers `audit-sync` as optional maintenance.

## Inspection state and resume

Throughout the workflow the agent keeps `<repo>/.ono/state.json` up to date via the internal `inspection-state` skill (after every stage and every Stage 3 loop step). This file is the plugin's committed, portable orchestration memory: plugin/schema versions, completed stages, a reconciled snapshot of the `AUDIT.md` topics, and a `resume` pointer. Because of it, restarting `/inspect` (or running `/inspect-approve`) after an interruption continues exactly where you left off — the reviewed Draft still awaiting approval, or the next topic to break down — instead of re-deriving progress from files. `AUDIT.md` stays the source of truth; `state.json` only mirrors it. The developer never invokes `inspection-state` directly; it is internal infrastructure. Commit `.ono/state.json` so progress is shared with the team.

## Maintenance (outside the workflow)

`audit-sync` is a documentation-maintenance tool, not a workflow stage (`workflowRole: maintenance` in the registry). Run it on demand with `/inspect-sync` once one or more topics are `Approved`. It reads the currently-`Approved` topics and regenerates the two managed blocks in `CLAUDE.md` (Caution Areas and Important Files), verifies `AUDIT.md` consistency, detects drift, repairs references inside its managed blocks, and reports. It **never** marks anything `Approved` and treats `AUDIT.md` as read-only. Re-run it whenever more topics are approved.

## Shortcuts onto the same steps

`/inspect-status` can be run at any point and never changes state. `/inspect-approve` performs whichever gate is currently open — the non-repeatable stage gates (step 3 or 5) or the Stage 3 review gate (finalize the Draft via `audit-approve`, then break down the next topic). `/inspect-topic <name>` performs one `audit-breakdown` cycle directly for a specific topic without first running `/inspect`. `/inspect-sync` runs maintenance. All read the same `skills/registry.json` and `AUDIT.md` the guided flow uses, so state never diverges between commands.

## What never happens

- The agent never reads or writes repository source files directly.
- No inspection stage runs without the developer having approved the previous one.
- No second `audit-breakdown` Draft is generated without a developer review of the previous Draft in between.
- Only `audit-approve` ever changes a topic to `Approved`, and only on explicit developer approval. `audit-breakdown` and `audit-sync` never mark anything `Approved`.
- `audit-sync` never writes `AUDIT.md` and never runs as part of the normal workflow.
- No skill creates implementation plans, Jira tasks, or source-code patches — that is out of scope for the entire plugin by design.
