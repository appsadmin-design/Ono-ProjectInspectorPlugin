---
description: Directly request breakdown of a specific audit topic (or the next pending one), skipping the general workflow narrative.
argument-hint: "[topic-name] [repository-path]"
---

Invoke the `project-inspector` agent in **`targeted` mode** (see the agent's "Invocation Modes" section in `agents/project-inspector.md`).

Treat the first argument as the topic name if provided (e.g. "Managers and Singletons"). If it's omitted, tell the agent to default to the first `Pending Breakdown` topic in `AUDIT.md`. Treat any remaining argument as the repository path if the agent needs it.

This command does not bypass the underlying skill's own confirmation step — it only removes the need to run `/inspect` and answer "which topic" conversationally first. The agent still checks that `AUDIT.md` exists before proceeding, and still stops for developer approval after the one topic is processed.

$ARGUMENTS
