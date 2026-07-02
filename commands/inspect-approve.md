---
description: Approve continuing past the current Ono Project Inspector approval gate (advance one stage, or process one more topic).
argument-hint: "[repository-path]"
---

Invoke the `project-inspector` agent in **`resume` mode** (see the agent's "Invocation Modes" section in `agents/project-inspector.md`).

This is the explicit, discoverable equivalent of replying "approved, continue" in conversation. It advances the workflow by exactly one step: the next unblocked stage, or one more cycle if the last completed stage was repeatable (e.g. one more `audit-breakdown` topic). If an argument was provided, treat it as the repository path in case conversation context was lost; otherwise rely on context from the current session.

This command never advances more than one step, even if multiple stages could technically run.

$ARGUMENTS
