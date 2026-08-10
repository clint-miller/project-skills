# Director Workflow

Shared control-plane workflow for interacting with Director and managed project repositories.

## What it standardizes

- whole-queue status;
- individual project status;
- adding explicit work;
- deriving needed work from a project's own status/roadmap and live evidence;
- planning next work without automatically queuing it;
- queue control and reprioritization;
- full-queue or project-level reconciliation;
- reporting only items that genuinely require Clint.

## Commands

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

Natural-language equivalents are intentionally supported. Examples include `What is the Director status?`, `What is going on with Futures?`, `Add this to Futures: ...`, `Look at Futures and add whatever it actually needs next`, `Make Futures the top priority`, and `What needs me?`.

## Core architecture

The skill keeps four kinds of state separate:

- **managed repository** — project truth, roadmap, decisions, blockers, resumable state;
- **Director** — orchestration/queue/ownership truth;
- **Git/PR/CI/deployment systems** — proof of work;
- **conversation** — current user intent, not durable memory.

## Required lifecycle

Material project work must follow:

```text
Inspect -> Work -> Verify -> Synchronize repo state -> Handoff
```

Repository status/roadmap/checkpoint documentation must reflect material project changes before a successful handoff. If the needed synchronization cannot be performed, the agent must report that explicitly rather than claiming a clean handoff.

`SKILL.md` is authoritative.