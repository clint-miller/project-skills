# Project Skills

Shared, project-agnostic skills for use across project repositories.

This repository is intended to be included in other repositories as a Git submodule so reusable workflows can be maintained once and consumed consistently everywhere.

## Repository layout

Each top-level skill keeps its authoritative entry point in `SKILL.md`, but a skill may contain narrowly scoped role entry points when doing so avoids forcing unrelated workflow policy into every agent.

```text
project-skills/
├── README.md
├── agent-reporting/
│   ├── README.md
│   └── SKILL.md                         # compatibility shim
├── director-workflow/
│   ├── README.md
│   ├── SKILL.md                         # general/interactive Director workflow
│   ├── director-agent/
│   │   └── SKILL.md                     # autonomous Director entry point
│   ├── repo-agent/
│   │   └── SKILL.md                     # autonomous Repo entry point
│   └── shared/
│       └── reporting-handoff.md         # role-neutral report/handoff contract
└── pw-workflow/
    ├── README.md
    └── SKILL.md
```

New reusable top-level skills should normally follow:

```text
<skill-name>/
├── README.md
└── SKILL.md
```

Role-specific nested entry points are appropriate only when they keep reusable process modular without creating competing sources of truth.

## Add to another project as a submodule

From the root of the consuming repository:

```bash
git submodule add https://github.com/clint-miller/project-skills.git .project-skills
git commit -m "Add shared project skills submodule"
```

The recommended mount point is `.project-skills/`. Using the same location in every project keeps project instructions portable.

The developer, automation runner, or agent cloning a consuming repository must be able to access and initialize `clint-miller/project-skills`.

## Clone a project that uses the submodule

```bash
git clone --recurse-submodules <project-repository-url>
```

If the project was already cloned:

```bash
git submodule update --init --recursive
```

## Update shared skills in a consuming project

A submodule is pinned to an exact commit. Updating `project-skills` does not silently change consuming projects. This is intentional: each project gets reproducible behavior until its submodule pointer is deliberately advanced.

From the consuming project:

```bash
cd .project-skills
git checkout main
git pull --ff-only
cd ..
git add .project-skills
git commit -m "Update shared project skills"
```

## Required parent-project instruction

Adding the submodule only makes the files available. The parent project should explicitly tell ChatGPT/agents to consult the shared skills.

Recommended project instruction:

```text
Shared reusable skills are available in `.project-skills/`, which is a Git submodule of `https://github.com/clint-miller/project-skills`.

When a user request matches a shared skill or invokes one of its defined commands, read that skill's `.project-skills/<skill-name>/SKILL.md` before acting and follow it subject to higher-priority project and system instructions.

Do not modify files inside `.project-skills/` as part of normal project work. Changes to shared skills belong in the `clint-miller/project-skills` repository. The parent repository should only update the pinned submodule commit when adopting a newer shared-skills version.
```

Project-specific instructions remain authoritative for project-specific policy. Shared skills provide reusable workflow behavior without replacing local governance, safety rules, release policy, CI requirements, approval gates, scheduler authority, or production permissions.

## Available skills

### `agent-reporting`

Legacy-compatible entry point for the shared `STATUS / EVIDENCE / CHANGES / VALIDATION / ROADMAP / NEXT / BLOCKER / HANDOFF` envelope. During migration it redirects to `director-workflow/shared/reporting-handoff.md` so reporting semantics are shared by Director and Repo roles without duplicating execution policy.

See [`agent-reporting/README.md`](agent-reporting/README.md) and [`agent-reporting/SKILL.md`](agent-reporting/SKILL.md).

### `director-workflow`

Reusable Director workflow family.

- [`director-workflow/SKILL.md`](director-workflow/SKILL.md) remains the general/interactive Director control contract.
- [`director-workflow/director-agent/SKILL.md`](director-workflow/director-agent/SKILL.md) is the thin autonomous Director role entry point.
- [`director-workflow/repo-agent/SKILL.md`](director-workflow/repo-agent/SKILL.md) is the thin fenced Repo execution entry point.
- [`director-workflow/shared/reporting-handoff.md`](director-workflow/shared/reporting-handoff.md) contains the common role-neutral reporting/handoff semantics.

The separation keeps Director queue/control behavior out of Repo execution loads and keeps Repo implementation methodology out of Director control-plane loads.

Existing Director commands remain:

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

See [`director-workflow/README.md`](director-workflow/README.md).

### `pw-workflow`

Command-driven project workflow for bootstrap/status, task execution, PR review, and conditional merge operations.

Commands:

```text
pw-bootstrap [project_name]
pw-status
pw-review [pr]
pw-execute
pw-merge
pw-review-merge [pr]
pw-full [project_name]
```

See [`pw-workflow/README.md`](pw-workflow/README.md) and [`pw-workflow/SKILL.md`](pw-workflow/SKILL.md).

## Design rules

- Keep skills project-agnostic.
- Keep one canonical top-level skill per top-level directory.
- Use nested role entry points only to avoid unnecessary policy loading and preserve one shared source for common contracts.
- Use a distinctive command namespace when a skill exposes shorthand commands.
- Do not duplicate project-specific governance in shared skills.
- Treat each consuming project's submodule commit as its approved shared-skill version.
- Change shared skills in this repository, validate them, then deliberately update consuming projects to the desired commit.
