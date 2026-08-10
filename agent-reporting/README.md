# Agent Reporting

Shared concise handoff/reporting contract for autonomous agents.

Use `SKILL.md` when an agent finishes a bounded cycle, enters an asynchronous wait, becomes blocked/failed, hands work back to an orchestrator, or detects that its attempt has been superseded.

The standard envelope is always:

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

Use `None` rather than dropping headings. The envelope is intentionally terse; durable project status, roadmap, checkpoints, authority, and orchestration state remain in the target repository/runtime.

The current contract also makes these distinctions explicit:

- a completed bounded slice is not automatically a completed project;
- handoff reporting does not grant scheduler or worker-control authority;
- scheduler/UI telemetry is not automatically durable authority;
- ordinary CI waits, review returns, and later worker invocations preserve the existing execution lineage unless durable authority explicitly supersedes it;
- protected deployment success and owner/user-flow acceptance are separate validation facts;
- the `NEXT` action and `HANDOFF` resume point must be executable from durable state by a fresh authorized agent.

`SKILL.md` is authoritative.
