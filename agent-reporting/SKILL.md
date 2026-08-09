# Agent Reporting Skill

## Purpose

Use this skill whenever an autonomous agent completes, pauses, waits, fails, hands off, or is superseded during a bounded work cycle.

The goal is to make the result immediately actionable by another agent or control plane without conversational padding. The report is a transport envelope, not a replacement for durable project documentation.

## Required report order

Lead with state. Use these headings in this order:

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

Omit a heading only when it is genuinely not applicable. Never replace `BLOCKER` with vague uncertainty; use `None` when there is no blocker.

## Field contract

### STATUS

One or two lines describing the current bounded-work state. Prefer explicit lifecycle terms such as `running`, `waiting_ci`, `waiting_external`, `completed`, `blocked`, `failed`, `superseded`, or `late_exit` when the parent project defines them.

### EVIDENCE

List only durable evidence needed to verify the result: repository, branch, exact commit/head SHA, issue/PR, CI run/workflow, deployment/preview, artifact/checkpoint path, or other authoritative identifiers.

When CI is involved, include the exact candidate SHA the CI run is expected to validate.

### CHANGES

State what was actually changed during this cycle. Do not restate the objective. If no project mutation occurred, say `None`.

### VALIDATION

State what was tested or checked and the exact candidate/head it applied to. Separate passed, failed, queued, and in-progress validation. Never imply a run validated a newer or different SHA.

### ROADMAP

State only the remaining work relevant to the current objective. Durable roadmap/status truth belongs in the target repository and should be updated there when the cycle materially changes it.

### NEXT

Give exactly one recommended next action. It must be executable by a fresh agent from durable state.

### BLOCKER

Use `None`, or name the precise unresolved condition. Waiting for a normally progressing asynchronous dependency such as CI is not automatically a blocker.

### HANDOFF

Name the intended next role/worker and the durable resume point. For asynchronous waits, state who owns the wait and what event makes project work actionable again.

## Durable-state rule

The report is not authoritative project memory. Before handoff, persist substantive project state in the target repository or the parent orchestration runtime as required by that project's instructions.

A fresh agent with no chat history should be able to use the repository plus the identifiers in this report to continue safely.

## Waiting CI example

```text
STATUS
waiting_ci — implementation attempt completed and worker released.

EVIDENCE
repo=owner/project
branch=work/example/attempt-3
head=abc123
ci_run=123456789
expected_sha=abc123
checkpoint=.director/state.yaml

CHANGES
Implemented the assigned bounded change and pushed candidate abc123.

VALIDATION
Local checks passed on abc123. GitHub Actions run 123456789 is in_progress for abc123.

ROADMAP
Review CI result, then continue the objective according to the repository checkpoint.

NEXT
When run 123456789 becomes terminal, dispatch a fresh repo agent to review_ci_result against expected_sha abc123.

BLOCKER
None.

HANDOFF
Recovery Agent owns the CI wait; Repo Agent A/B resumes from the repository checkpoint after CI becomes terminal.
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
Successor attempt continues the objective.

NEXT
Director should reconcile any previously unrecorded commit on the old attempt branch if useful.

BLOCKER
None.

HANDOFF
No handoff from this stale attempt; stop after recording late-exit evidence.
```

## Style

- Answer first; no greetings, praise, throat-clearing, or narrative setup.
- Prefer identifiers and concrete facts over prose.
- Do not expose private chain-of-thought or speculative reasoning.
- Do not repeat unchanged repository documentation in the report.
- Keep the envelope concise enough to scan quickly, while including every identifier needed for verification/resume.
