---
description: Start (or resume) the complete Ono Project Inspector workflow on a local repository.
argument-hint: "[repository-path]"
---

Invoke the `project-inspector` agent in **`full` mode** (see the agent's "Invocation Modes" section in `agents/project-inspector.md`).

If an argument was provided, treat it as the target repository path and pass it to the agent as the initial repository location; the agent (via the `project-analysis` skill) will still confirm it before doing anything. If no argument was provided, let the agent's startup sequence ask for it.

This command performs no inspection logic itself — it only starts the agent in full-workflow mode. All orchestration, skill selection, and repository analysis happen inside `agents/project-inspector.md` and the skills it invokes. Use this command for the default, end-to-end experience; use `/inspect-status`, `/inspect-topic`, or `/inspect-approve` for narrower actions.

$ARGUMENTS
