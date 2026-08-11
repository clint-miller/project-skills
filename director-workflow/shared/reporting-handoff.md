# Shared Reporting and Handoff Contract

Use this contract at the end of a bounded Director or Repo cycle. It is deliberately role-neutral: durable authority and role-specific scheduler/merge/execution permissions remain in the consuming repository.

## Required envelope

Always report these headings in this order:

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

Use `None` when a heading has no applicable content.

## Semantics

### STATUS
State the bounded lifecycle condition, not vague activity. Examples: `running`, `returned_to_control_plane`, `waiting_ci`, `waiting_external`, `completed`, `blocked`, `failed`, `superseded`, `late_exit`.

### EVIDENCE
Include only identifiers needed to verify/resume: repository, work id, attempt/fence, branch/head SHA, issue/PR, exact CI run and expected SHA, deployment/artifact/checkpoint paths.

### CHANGES
State what actually changed in this cycle. If nothing durable changed, say `None`.

### VALIDATION
State exact checks and which candidate/head they validated. Keep queued/in-progress/failed/passed states distinct.

### ROADMAP
State remaining work relevant to the current objective. Material project truth belongs in repository-native status/roadmap/checkpoint state before a clean handoff.

### NEXT
Give exactly one executable next action that a fresh authorized agent can perform from durable state.

### BLOCKER
Use `None` or the precise unresolved condition. Normal asynchronous CI is not automatically a blocker. Human action is required only when durable governance/evidence genuinely requires it.

### HANDOFF
Name the next role/control-plane owner and durable resume point. A report does not grant scheduler authority and does not create a new attempt/fence/branch.

## Invariants

- Durable repository/runtime state is authoritative; the report only summarizes it.
- Completing one bounded slice does not imply whole-project completion.
- Material status/roadmap/decision changes must be synchronized before claiming a clean handoff.
- Ordinary review, CI waits, or a later worker invocation preserve the current lineage unless durable authority explicitly supersedes it.
- Scheduler/UI telemetry is evidence only unless the consuming project explicitly makes it authoritative.
- Do not expose private reasoning or secrets.

## Compatibility

`agent-reporting/SKILL.md` remains the legacy public entry point during migration. It redirects consumers to this contract while retaining the same eight-heading envelope and semantics.
