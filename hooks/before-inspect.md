# Hook: before-inspect

Type: agent-read checkpoint instruction.
Consumed by: `agents/project-inspector.md`, at the very start of the workflow, before any skill is invoked.

This is not a Claude Code `hooks.json` event. It is a plain checklist the orchestrating agent reads and follows as part of its own startup sequence.

## Checklist

1. Confirm a target repository path is known. If not, do not guess — let the first invoked skill's own "Step 1: Ask the Developer" collect it, or ask directly if the workflow needs it before stage selection.
2. Do not verify the repository by scanning its contents yourself — that is `project-analysis`'s job. Limit any check here to confirming the path was provided, not to inspecting what's inside it.
3. Check whether `skills/registry.json` is present and parses. If it is missing or invalid, stop and report the workflow cannot proceed — this file is required for orchestration.
4. Note which registry entries are `enabled: false` and mention them once, briefly, in the initial workflow plan so the developer knows they exist but won't run.
5. Do not ask the developer questions that a downstream skill will ask anyway (e.g. overwrite/update/preserve/version preferences for existing artifacts). Let the skill own its own intake questions.

## Why this exists

Keeps the agent from duplicating a skill's own intake logic, and gives a single place to add future global pre-flight checks (e.g. a workspace trust check) without touching the agent's core loop.
