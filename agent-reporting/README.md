# Agent Reporting

Shared concise handoff/reporting contract for autonomous agents.

Use `SKILL.md` when an agent finishes a bounded cycle, enters an asynchronous wait, becomes blocked/failed, hands work back to an orchestrator, or detects that its attempt has been superseded.

The standard envelope is:

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

The envelope is intentionally terse. Durable project status, roadmap, checkpoints, and architecture remain in the target repository; the report carries the exact evidence and resume information needed by the next agent/control plane.
