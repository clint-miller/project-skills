---
name: agent-reporting
description: Concise durable-state reporting contract for autonomous agents completing, pausing, waiting, failing, handing off, or exiting a bounded work cycle. Use it to make results verifiable and resumable without inventing authority, project completion, scheduler effects, or new execution lineages.
---

# Agent Reporting Skill

## Purpose

Use this skill whenever an autonomous agent completes, pauses, waits, fails, hands off, or is superseded during a bounded work cycle.

The goal is to make the result immediately actionable by another agent or control plane without conversational padding. The report is a transport envelope, not a replacement for durable project documentation or orchestration state.

## Authority rules

1. **Durable project/runtime state is authoritative.** The report summarizes it; the report does not create new authority.
2. **A completed bounded slice is not automatically a completed project.** Do not declare the managed project finished unless its authoritative project-native status/roadmap supports that conclusion.
3. **A handoff report does not grant scheduler or worker-control authority.** Follow the parent project's role boundaries. If the current role is not allowed to schedule/rearm another worker, persist the handoff durably and report that scheduler action belongs elsewhere.
4. **Do not invent a new execution lineage.** CI waits, control-plane review, or a later worker invocation do not by themselves require a new attempt/branch/fence. Preserve the current lineage unless durable authority explicitly supersedes it.
5. **Telemetry is not automatically authority.** UI labels, task status, timestamps, or other scheduler metadata should be reported as telemetry when useful, not treated as stronger than the project's declared durable source of truth.
6. **Never expose private chain-of-thought.** Report decisions, evidence, and outcomes only.

## Required report order

Use every heading, in this order:

```text
STATUS
EVIDENCE
CHANGES
VALIDATION
ROADMAP
NEXT
BLOCKER
HANDOFF
```

Use `None` when a field has no applicable content. Keeping all headings makes reports machine- and human-scannable across projects.

## Field contract

### STATUS

One or two lines describing the current bounded-work state. Prefer explicit lifecycle terms such as `running`, `returned_to_control_plane`, `ci_observing`, `waiting_ci`, `waiting_external`, `completed`, `blocked`, `failed`, `superseded`, or `late_exit` when the parent project defines them.

Distinguish the bounded work object's state from the overall project's state when that matters. For example, `completed — Today acceptance slice complete; Atlas Dashboard remains active` is preferable to an ambiguous `completed`.

### EVIDENCE

List only durable evidence needed to verify or resume the result: repository, work/objective id, attempt/fence when applicable, branch, exact commit/head SHA, issue/PR, CI run/workflow, deployment/preview, artifact/checkpoint path, or other authoritative identifiers.

When CI is involved, include the exact candidate SHA the run is expected to validate. Do not imply that a run for one SHA validates another.

When scheduler/task telemetry is materially useful, label it as observed telemetry unless the parent project explicitly makes it authoritative.

### CHANGES

State what was actually changed during this cycle. Do not restate the objective. If no project/runtime mutation occurred, say `None`.

Do not claim a scheduler mutation, merge, deployment, notification, or other external effect unless it actually occurred and was verified.

### VALIDATION

State what was tested or checked and the exact candidate/head it applied to. Separate passed, failed, queued, in-progress, and not-run validation.

For protected/manual acceptance, distinguish automated deployment success from owner/user-flow acceptance. One does not imply the other.

### ROADMAP

State only the remaining work relevant to the current objective and the next project-level lane when it is material to continuation.

Durable roadmap/status truth belongs in the target repository and **must be updated there before a clean successful handoff whenever the cycle materially changed project truth**. Never infer whole-project completion merely because the current issue, PR, or bounded work object completed.

If the project defines a native status/roadmap workflow, use that authoritative workflow before making a project-level completion or next-lane claim.

### NEXT

Give exactly one recommended next action. It must be executable by a fresh authorized agent from durable state.

Preserve the current attempt/lineage when the next action is ordinary review, CI-result handling, quick bounded repair, or continuation and the parent project has not superseded it.

### BLOCKER

Use `None`, or name the precise unresolved condition and, when applicable, the exact owner/external action required.

Waiting for a normally progressing asynchronous dependency such as CI is not automatically a blocker. A human gate is a blocker only when the parent project's governance or current evidence genuinely requires that human action before progress can continue.

If material project truth changed but required repository status/roadmap/checkpoint state could not be synchronized, report that exact unsynchronized state here. Do not hide it behind a successful handoff.

### HANDOFF

Name the intended next role/worker or control-plane owner and the durable resume point.

A handoff description is not permission to mutate a scheduler. Follow the parent project's authority model:

- if the current role has no scheduler authority, state the durable return target/resume point and stop;
- if another control-plane role owns future scheduling, name that role;
- if an authorized scheduler effect already occurred, report the verified target/effect without implying more than was actually done.

For asynchronous waits, state who owns monitoring and what terminal event makes the existing project lineage actionable again.

A clean successful `HANDOFF` requires that material project truth already be synchronized into the repository or other project-native durable state required by project instructions. If synchronization is required but incomplete, state the incomplete handoff and corresponding blocker instead of claiming that a fresh agent can safely resume.

## Durable-state rule

The report is not authoritative project memory.

Before handoff, persist substantive project state in the target repository or parent orchestration runtime as required by that project's instructions. When the cycle materially changes status, roadmap, next steps, blockers, locked decisions, validation state, or resumable technical state, synchronize the corresponding project-native durable sources before reporting a clean successful handoff.

At minimum, when applicable:

1. update current status;
2. mark completed/obsolete next steps so they are not repeated;
3. record new remaining work and dependency order;
4. record blockers/waits and who or what resolves them;
5. preserve material decisions/constraints discovered during the cycle;
6. record exact verification/resume evidence.

Do not create documentation churn for immaterial changes.

If required durable synchronization is impossible because of missing authority, unavailable source material, write failure, conflicting project instructions, or another real constraint, report the exact condition under `BLOCKER` and do not present the result as a clean successful handoff.

A fresh agent with no chat history should be able to use the repository plus the identifiers in this report to continue safely.

## Execution-lineage rule

Treat an execution attempt/branch/fence as a durable lineage when the parent project defines one.

Ordinary events such as these do **not** by themselves create a new lineage:

- a Scheduled Task invocation ends;
- control returns to a Director/reviewer;
- CI is queued or running;
- CI becomes terminal;
- a bounded same-objective defect is found;
- a different reusable worker later resumes the same authorized attempt.

Only report a new attempt/branch/fence when durable project authority actually created or superseded one.

## Waiting CI example

```text
STATUS
waiting_ci — implementation is checkpointed; the existing execution lineage is detached while CI runs.

EVIDENCE
repo=owner/project
work_id=example-v1
attempt_id=3
fence_token=3
branch=work/example-v1/attempt-3
head=abc123
ci_run=123456789
expected_sha=abc123
checkpoint=.director/state.yaml

CHANGES
Implemented the assigned bounded change and pushed candidate abc123.

VALIDATION
Local checks passed on abc123. GitHub Actions run 123456789 is in_progress for expected_sha abc123.

ROADMAP
The current bounded objective remains active. Repository status/roadmap/checkpoint state records the CI wait and expected SHA abc123. Review the exact CI result before any merge or further implementation decision.

NEXT
When run 123456789 becomes terminal, reconcile it against expected_sha abc123 and resume the same attempt-3 lineage if authority is unchanged.

BLOCKER
None.

HANDOFF
The project's designated async/control-plane role owns monitoring. The next authorized repo invocation resumes from attempt 3 and the durable checkpoint; this report does not itself schedule that invocation.
```

## Scheduler-free repo return example

```text
STATUS
returned_to_control_plane — bounded repo work is checkpointed and this worker is exiting.

EVIDENCE
repo=owner/project
work_id=example-v1
attempt_id=3
branch=work/example-v1/attempt-3
head=def456
checkpoint=state/work/example-v1.json

CHANGES
Completed the assigned bounded implementation and synchronized repository status/roadmap state before return.

VALIDATION
Focused tests passed on def456. No additional validation is claimed.

ROADMAP
Repository project truth reflects the completed slice and remaining work. Director/control-plane reconciliation determines the next project slice from that durable state.

NEXT
Control plane should reconcile the persisted return state and choose the next authorized action.

BLOCKER
None.

HANDOFF
Returned durably to the control plane at state/work/example-v1.json. This repo role performs no scheduler mutation.
```

## Superseded/late-exit example

```text
STATUS
late_exit — this attempt was superseded and no longer has write authority.

EVIDENCE
attempt_id=3
observed_fence_token=4
branch=work/example/attempt-3
branch_head=def456

CHANGES
No new project mutation after detecting supersession.

VALIDATION
Confirmed durable runtime assigns a newer attempt/fence token.

ROADMAP
The successor lineage continues the bounded objective; this stale attempt must not reclaim ownership.

NEXT
Control plane should reconcile any useful unrecorded evidence from the old branch without restoring its write authority.

BLOCKER
None.

HANDOFF
None from this stale attempt. Stop after recording late-exit evidence.
```

## Style

- Answer first; no greetings, praise, throat-clearing, or narrative setup.
- Prefer identifiers and concrete facts over prose.
- Keep lifecycle state, project state, scheduler telemetry, and durable authority distinct.
- Do not repeat unchanged repository documentation in the report.
- Keep the envelope concise enough to scan quickly while including every identifier needed for verification and resume.
