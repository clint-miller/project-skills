# Director Workflow

Shared workflow family for Director orchestration, fenced Repo execution, and common reporting/handoff semantics.

## Role-scoped package layout

```text
director-workflow/
├── README.md
├── SKILL.md                         # interactive/general Director workflow compatibility entry point
├── director-agent/
│   └── SKILL.md                     # autonomous Director role entry point
├── repo-agent/
│   └── SKILL.md                     # autonomous fenced Repo role entry point
└── shared/
    └── reporting-handoff.md         # role-neutral durable report/handoff contract
```

The role entry points are intentionally thin. A Director agent does not need to load Repo execution methodology, and a Repo agent does not need to load Director queue-control methodology.

`agent-reporting/SKILL.md` remains available as a compatibility shim and redirects to the shared reporting/handoff contract during migration.

## Director operations

The existing top-level `SKILL.md` continues to standardize human/interactive Director operations including:

```text
director-status
director-project-status <project>
director-add-work <project>: <request>
director-add-work <project> --derive
director-plan-work <project>
director-control <instruction>
director-reconcile
director-reconcile <project>
director-needs-me
```

Natural-language equivalents remain supported.

For an autonomous Director worker, load `director-agent/SKILL.md`; it references the top-level Director orchestration contract plus only the shared handoff contract.

For an autonomous Repo worker, load `repo-agent/SKILL.md`; it owns fenced assignment verification and bounded repository execution without inheriting Director queue/scheduler policy.

## Core architecture

The workflow family keeps these authority domains separate:

- **managed repository** — project truth, roadmap, decisions, blockers, resumable state;
- **Director** — orchestration/queue/ownership truth;
- **Git/issue/PR/CI/deployment systems** — proof of work;
- **conversation** — current user intent, not durable memory.

Material work follows:

```text
Inspect -> Work -> Verify -> Synchronize repo state -> Handoff
```

Reusable process belongs in shared skills. Consuming repositories retain local authority for governance, permissions, scheduler topology, safety boundaries, CI, merge policy, and production actions.

Pinned `.project-skills` consumers must deliberately advance their submodule revision before receiving this workflow family.
