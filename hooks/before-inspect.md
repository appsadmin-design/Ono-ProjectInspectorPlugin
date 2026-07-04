# Hook: before-inspect

Type: agent-read checkpoint instruction.
Consumed by: `agents/project-inspector.md`, at the very start of the workflow, before any skill is invoked.

This is not a Claude Code `hooks.json` event. It is a plain checklist the orchestrating agent reads and follows as part of its own startup sequence.

## Checklist

1. Confirm a target repository path is known. If not, do not guess — let the first invoked skill's own "Step 1: Ask the Developer" collect it, or ask directly if the workflow needs it before stage selection.
2. Do not verify the repository by scanning its contents yourself — that is `project-analysis`'s job. Limit any check here to confirming the path was provided, not to inspecting what's inside it.
3. Check whether `skills/registry.json` is present and parses. If it is missing or invalid, stop and report the workflow cannot proceed — this file is required for orchestration.
4. Note which registry entries are `enabled: false` and mention them once, briefly, in the initial workflow plan so the developer knows they exist but won't run.
5. **Inspection state (read-only detect — write nothing yet).** Once the repository path is known, invoke the internal `inspection-state` skill (`detect`), which does not write. Then:
   - Do **not** `init` here. State is first written later (via the post-stage `sync`) only once the developer chooses to start or continue in the Smart Startup Decision, so that choosing "leave everything unchanged" leaves the repo untouched.
   - If it reports `needsMigration: true` for an existing inspection, invoke `inspection-state` (`migrate`) before continuing that inspection.
   - If it reports `versionMismatch: true`, tell the developer the state was last written by a different plugin version (report both versions) before continuing — do not silently proceed.
   - Hand the `detect` result — `inspected`, completed/current stages, counts, and the `resume` pointer — to the agent's Smart Startup Decision, which presents the status and choices rather than re-deriving progress from files. This is what makes resume-after-interruption reliable.
   `inspection-state` is internal, deterministic, and never writes source files; `detect`/`migrate` are the only pre-flight skill invocations.
6. Do not ask the developer questions that a downstream skill will ask anyway (e.g. overwrite/update/preserve/version preferences for existing artifacts). Let the skill own its own intake questions.

## Why this exists

Keeps the agent from duplicating a skill's own intake logic, and gives a single place to run global pre-flight checks — now including inspection-state detection, version-mismatch/migration handling, and resume — without touching the agent's core loop.
