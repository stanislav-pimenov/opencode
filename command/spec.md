---
description: Turn a rough requirement into a checkable spec and implementation plan.
agent: plan
---

Requirement from the user:

$ARGUMENTS

Produce a compact spec in markdown, no code changes:

1. **Problem** — what the user cannot do today, in one paragraph.
2. **Scope** — in scope / explicitly out of scope.
3. **Acceptance criteria** — numbered, each one objectively checkable by a command, a test, or a concrete user action.
4. **Affected areas** — files, modules, APIs, DB/schema, config; read the code before listing them.
5. **Design** — the smallest approach that satisfies the criteria; call out alternatives only when a real trade-off exists.
6. **Test plan** — which tests to add or change, and how to run them (use the project's real commands).
7. **Risks & open questions** — max 5, phrased as questions for the user.

If the requirement is ambiguous, state the ambiguity in **Open questions** and state the assumption you would proceed with. Do not implement anything.
