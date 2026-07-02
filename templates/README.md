# templates/

Intentionally empty for now.

Both vendored skills (`project-analysis`, `audit-breakdown`) explicitly embed their own markdown templates inside their `SKILL.md` files and state they must not depend on external template files. Wiring this folder into either skill would contradict their documented output contracts, so it is left unused by the current workflow.

This folder is reserved for future skills that may prefer externalized templates (e.g. a skill that supports multiple output formats). If a future skill uses it, document the convention here at that time.
