---
description: Build, test, lint and check acceptance criteria using the verifier subagent.
agent: build
---

Delegate verification to the `verifier` subagent using the task tool.

Acceptance criteria (may be empty — then use the criteria from the recent spec or infer them from the change):

$ARGUMENTS

In the delegated prompt, include:
- the working directory,
- the exact acceptance criteria (or "infer from the current diff"),
- an instruction to find real build/test/lint commands in AGENTS.md / README / Makefile / CI config,
- an instruction to report in the criterion → evidence table format and to mark anything unproven as `unverified`.

After the subagent returns, summarize its verdict in at most 5 lines. Do not fix failures unless the user asks.
