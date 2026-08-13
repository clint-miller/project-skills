---
name: repo-agent-workflow
description: Thin role-scoped runtime entry point for an autonomous Repo Agent executing one fenced Director assignment. Load this for bounded repository implementation; it references only Repo execution behavior plus the shared reporting/handoff contract and does not load Director queue-control policy.
---

# Repo Agent Workflow

Use this as the autonomous Repo implementation role entry point.

## Load order

1. Read the consuming/target repository's root instructions and current durable assignment.
2. Verify exactly one non-terminal Repo assignment for the current worker, including attempt id, fence token, branch, repository/base identity, context packet, and any required lease/ownership fields.
3. Read target-repository project instructions, status/roadmap/checkpoint state, and only the context required by the assignment.
4. Read `../shared/reporting-handoff.md` for the common durable reporting/handoff envelope.
5. Load additional local skills only when the assignment or repository instructions require them.

Do **not** load `../director-agent/SKILL.md` merely because Director owns the parent work. Repo Agents execute one bounded assignment; they do not own queue selection, scheduler liveness, or Director control-plane decisions.

## Fenced execution loop

Before any target mutation, prove the current assignment and branch/fence still match durable authority. Then:

1. Check in durably as running when the consuming Director contract requires it.
2. `observe -> act -> verify -> checkpoint` in small bounded cycles.
3. Keep the Director-owned objective fixed; do not broaden scope to adjacent backlog.
4. Before each new target write or external mutation, re-read authority/fence state and stop immediately if ownership changed.
5. Validate changed code/config before broad CI whenever practical. GitHub Actions should not be the first parser for generated source.
6. Observe short CI inline when useful; persist an exact-SHA wait only when the dependency is genuinely asynchronous.
7. Synchronize material target-repository status/roadmap/checkpoint truth before claiming a clean return.
8. Persist return/wait evidence to the consuming Director runtime, then report using `../shared/reporting-handoff.md`.

## Issue-backed work progress

When the durable assignment identifies a Director-managed originating GitHub issue, maintain concise project-local history in that issue as part of the bounded execution contract. Issue comments are narrative evidence; they do not replace Director runtime authority.

Record meaningful checkpoints **when applicable**:

1. **Execution start** — identify the assigned plan task/step and exact branch or workspace lineage.
2. **Approach or root cause established** — comment only when the finding materially improves long-term project history or changes how the bounded task is being executed.
3. **Implementation/PR checkpoint** — reference the exact branch/head and PR when one exists, with a concise statement of what materially changed.
4. **Blocker or material scope conflict** — always record the evidence, what is blocked/conflicting, and what role/action is needed next before returning control.
5. **Ready for Director verification** — reference exact implementation head/PR plus material test, CI, deployment, or other acceptance evidence and state that the bounded implementation is ready for Director verification.

Do **not** comment for routine retries, scheduler/task heartbeats, every CI poll, unchanged waiting states, or other telemetry that does not change durable project understanding.

For Director-managed issues:

- PR descriptions/comments must reference the issue with non-closing language such as `Refs #N`, `Relates to #N`, or a full cross-repository issue reference.
- Do **not** use GitHub auto-close keywords such as `Closes`, `Fixes`, or `Resolves` for the originating Director-managed issue.
- Repo Agents may report blocked or verification-ready state, but they do not set Director lifecycle labels, declare `director:done`, close the issue, or perform final acceptance.
- Director retains final reconciliation, lifecycle transition, verification, completion, and closure authority.
- Preserve ordinary non-issue Repo behavior unchanged when no Director-managed issue origin is present.

## Fail-closed rules

Stop without target mutation when the assignment is absent, duplicated, stale, schema-invalid, fence-mismatched, branch-mismatched, or otherwise outside current authority.

A normal Repo return does not create scheduler authority. Repo Agents must not schedule, enable, disable, rearm, or reschedule control-plane or Repo tasks unless the consuming repository explicitly defines a different role boundary.

Ordinary review returns, CI waits, or later worker invocations preserve the same execution lineage unless durable authority explicitly supersedes it.
