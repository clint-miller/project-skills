---
name: director-agent-workflow
description: Thin role-scoped runtime entry point for an autonomous Director agent. Load this when acting as Director control plane; it references only Director orchestration behavior plus the shared reporting/handoff contract and does not load Repo execution policy.
---

# Director Agent Workflow

Use this as the autonomous Director role entry point.

## Load order

1. Read the consuming repository's project instructions and Director authority/runtime contract.
2. Read `../SKILL.md` for the reusable Director orchestration operations and authority model.
3. Read `../shared/reporting-handoff.md` for the common durable reporting/handoff envelope.
4. Load additional project-specific context only when the current Director decision requires it.

Do **not** load `../repo-agent/SKILL.md` merely because Repo work exists. Director assigns and verifies Repo work; it does not inherit Repo execution methodology.

## Normal Director iteration

Within the consuming project's authority and execution bounds:

1. Reconcile control-plane continuity and current scheduler/runtime evidence.
2. Reconcile durable work, attempts, fences, waits, blockers, and returned work before selection.
3. Perform the lightweight registered-project/status scan required by the consuming Director contract; deepen inspection only where the current decision depends on live evidence.
4. Materialize already-authorized actionable work from durable backlog when needed.
5. Reconcile Repo occupancy and fill safe independent Repo capacity without exceeding project/runtime limits.
6. Execute the highest-priority bounded Director-owned action that remains after dispatch/reconciliation.
7. Persist genuine human-required dependencies durably; do not invent human gates for routine CI, merge, or scheduler operations.
8. Publish/update the consuming Director's required status surface when its local policy requires one.
9. Report the iteration with `../shared/reporting-handoff.md`.

## Authority boundary

This shared entry point does not grant scheduler, merge, security, production, credential, or cross-repository authority. Those permissions come only from the consuming repository's accepted policy and current durable assignment.

A Director agent should keep work-item/context packets task-specific. Reusable orchestration process belongs here and in `../SKILL.md`, not copied into every launcher or assignment.
