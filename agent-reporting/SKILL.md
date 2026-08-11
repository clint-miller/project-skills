---
name: agent-reporting
description: Compatibility entry point for the shared autonomous-agent reporting and handoff contract. Existing consumers may continue loading this path during migration; canonical shared semantics live under director-workflow/shared/reporting-handoff.md.
---

# Agent Reporting Compatibility Shim

This path is preserved for existing consumers.

Before reporting a bounded Director or Repo cycle, read:

`../director-workflow/shared/reporting-handoff.md`

That shared contract is canonical for the reporting/handoff envelope during the role-scoped workflow migration.

The required headings remain unchanged and must always appear in this order:

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

Compatibility guarantees:

- durable repository/runtime state remains authoritative;
- a completed bounded slice is not automatically a completed project;
- reporting does not grant scheduler, merge, or worker-control authority;
- ordinary CI waits/review returns preserve the existing execution lineage unless durable authority supersedes it;
- material project truth must be synchronized before a clean handoff;
- exact branch/SHA/PR/CI evidence is preferred over activity telemetry.

Do not copy role-specific Director or Repo execution methodology into this shim. Director execution belongs in `../director-workflow/director-agent/SKILL.md`; Repo execution belongs in `../director-workflow/repo-agent/SKILL.md`.
