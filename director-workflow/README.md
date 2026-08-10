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
- reporting only items that genuinely require Clint;
- submitting Director work while remaining inside a managed project's chat/repository scope.

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

## Use from a managed project chat

When a chat is already scoped to a managed project, Director control-plane submission for that same project does not switch the project's working scope.

Examples:

```text
Add this to Director: <request>
Queue this for later: <request>
Add whatever this project needs next to Director.
What does Director have queued for this project?
```

The current project is inferred unless the user names another target. The workflow writes only the minimum required orchestration state; it does not begin unrelated development in the Director repository.

Existing consuming repositories use pinned `.project-skills` submodule commits. They must deliberately advance that pointer before they receive a newer shared-skill version.

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