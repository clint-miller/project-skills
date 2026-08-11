# Agent Reporting

Compatibility entry point for the shared reporting/handoff contract used by autonomous Director and Repo workflows.

Existing consumers may continue loading `agent-reporting/SKILL.md`. During migration that file redirects to the canonical shared contract at:

```text
director-workflow/shared/reporting-handoff.md
```

The standard envelope remains unchanged:

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

Use `None` rather than dropping headings. Durable project/runtime state remains authoritative; reporting summarizes it and does not create scheduler, merge, worker-control, or execution-lineage authority.

Role-specific execution now lives separately:

- `director-workflow/director-agent/SKILL.md` — Director orchestration entry point.
- `director-workflow/repo-agent/SKILL.md` — fenced Repo execution entry point.

This directory remains available as a compatibility shim until consuming repositories deliberately migrate their pinned shared-skill revision and loader paths.
