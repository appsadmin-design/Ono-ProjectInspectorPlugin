# Hook: after-audit-sync

Type: agent-read checkpoint instruction.
Consumed by: `agents/project-inspector.md`, immediately after each `audit-sync` run reports completion.

Not a Claude Code `hooks.json` event — a checklist the agent follows itself.

## Checklist

1. Confirm the skill only marked as `Approved` the specific topics the developer explicitly named as approved in this conversation. If any topic was flipped to `Approved` that the developer did not name, flag it as a contract violation — only the developer approves audits.
2. Confirm the skill modified `CLAUDE.md` only inside the two managed blocks (`audit-sync:caution-areas` and `audit-sync:important-files`) and `AUDIT.md` only in the `Status` column of approved rows. If it reported writing anywhere else, flag it and do not attempt to fix it yourself.
3. Confirm the managed blocks were regenerated from ALL currently-Approved topics, not just the newly approved ones — this is what keeps re-runs idempotent. If the report suggests entries were appended rather than replaced, flag possible duplication.
4. If the skill reported that an approved topic's audit file was missing or unreadable, confirm it left that topic as `Draft` and reported the problem rather than guessing findings.
5. This stage has `requiresApproval: true` and is repeatable — do not auto-run it again. Only re-run when the developer approves additional topics later.
6. In your stage-transition report, tell the developer how many topics are now `Approved` and how many remain `Draft` or `Pending Breakdown`, so they know whether more sync/breakdown cycles are worthwhile.

## Why this exists

`audit-sync` is the only stage that writes back into `CLAUDE.md`, and it does so through fragile marker-delimited blocks driven by developer approval. This hook is the enforcement point that keeps approval developer-owned, keeps edits inside the managed markers, and verifies the idempotent full-regeneration contract independently of the LLM-driven edit.
