# Plugin Workflow

## Commands

The plugin exposes four commands, all thin wrappers around `agents/project-inspector.md`, which dispatches on the mode each command declares (see the agent's "Invocation Modes" section):

- `/inspect [repo-path]` — `full` mode. Runs the guided end-to-end workflow described below.
- `/inspect-status [repo-path]` — `status` mode. Read-only snapshot of progress; invokes no skill.
- `/inspect-topic [topic-name] [repo-path]` — `targeted` mode. Runs one `audit-breakdown` cycle directly for the named (or next pending) topic.
- `/inspect-approve [repo-path]` — `resume` mode. Advances exactly one step past the current approval gate.

A developer never needs to know a skill's name to use the plugin — every command already knows which skill(s) it needs and lets the agent handle sequencing.

## Running the guided workflow

```text
/inspect /path/to/repository
```

or just `/inspect`, and the agent will ask for the repository path.

## Step-by-step

1. **Startup** — the agent reads `skills/registry.json` and `hooks/before-inspect.md`, then reports the planned stage order (and any disabled skills, if the registry ever has one).
2. **Stage 1 — project-analysis** — the agent checks no prerequisites are needed, invokes `project-analysis`, which asks its own intake questions, confirms, then writes `CLAUDE.md` and `AUDIT.md` only. The agent runs `hooks/after-project-analysis.md`, reports the topic count, and stops for approval.
3. **Developer reviews** `CLAUDE.md` and `AUDIT.md`, then approves continuing.
4. **Stage 2 — project-docs** — the agent confirms `CLAUDE.md` exists, invokes `project-docs`, which reads the existing context, inspects source read-only, and writes the four `docs/project/*.md` files (overview, components, patterns, integrations). It never touches `CLAUDE.md`, `AUDIT.md`, or `audits/`. The agent runs `hooks/after-project-docs.md` and stops for approval.
5. **Developer reviews** the `docs/project/` files, then approves continuing.
6. **Stage 3 — audit-breakdown (cycle 1)** — the agent confirms `AUDIT.md` exists, invokes `audit-breakdown`, which asks the developer which `Pending Breakdown` topic to process (default: first), confirms, then writes exactly one `audits/<slug>/<slug>-audit.md` and updates one row in `AUDIT.md`. The agent runs `hooks/after-audit-breakdown.md` (deterministic slug/status check), reports how many topics remain, and stops for approval.
7. **Developer reviews** the Draft audit file and either asks to break down another topic, or names the topic(s) they approve.
8. **Stage 3 — audit-breakdown (cycle 2, 3, ...)** — repeats step 6 for each subsequent approval, one topic per cycle, until no `Pending Breakdown` topics remain or the developer stops.
9. **Stage 4 — audit-sync** — when the developer explicitly names one or more Draft topics as approved, the agent invokes `audit-sync`, which flips those rows to `Approved` in `AUDIT.md` and regenerates the two managed blocks in `CLAUDE.md` from all currently-Approved topics. The agent runs `hooks/after-audit-sync.md`. This stage is repeatable — run it again whenever more topics are approved.

## Shortcuts onto the same steps

`/inspect-status` can be run at any point in the above and never changes state. `/inspect-approve` performs exactly step 3 or step 5 above (whichever gate is currently open) without needing the developer to type free text. `/inspect-topic <name>` performs one cycle of step 4/6 directly, for a specific topic, without first running `/inspect`. All three read the same `skills/registry.json` and `AUDIT.md` the guided flow uses, so state never diverges between commands.

## What never happens

- The agent never reads or writes repository source files directly.
- No stage runs without the developer having approved the previous one (`requiresApproval` is true for every stage).
- `audit-sync` never marks a topic `Approved` unless the developer explicitly named it as approved.
- `audit-breakdown` never processes more than one topic per approval.
- No skill creates implementation plans, Jira tasks, or source-code patches — that is out of scope for the entire plugin by design.
