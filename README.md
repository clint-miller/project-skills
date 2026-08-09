# Project Skills

Shared, project-agnostic skills for use across project repositories.

This repository is intended to be included in other repositories as a Git submodule so reusable workflows can be maintained once and consumed consistently everywhere.

## Repository layout

Each skill lives in its own top-level directory:

```text
project-skills/
├── README.md
├── agent-reporting/
│   ├── README.md
│   └── SKILL.md
└── pw-workflow/
    ├── README.md
    └── SKILL.md
```

New reusable skills should follow the same pattern:

```text
<skill-name>/
├── README.md
└── SKILL.md
```

`SKILL.md` is the authoritative skill definition. The skill-local `README.md` is human-facing documentation.

## Add to another project as a submodule

From the root of the consuming repository:

```bash
git submodule add https://github.com/clint-miller/project-skills.git .project-skills
git commit -m "Add shared project skills submodule"
```

The recommended mount point is `.project-skills/`. Using the same location in every project keeps project instructions portable.

Because this repository is private, the developer, automation runner, or agent cloning a consuming repository must also have access to `clint-miller/project-skills`.

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

Project-specific instructions remain authoritative for project-specific policy. Shared skills should provide reusable workflow behavior without replacing local governance, safety rules, release policy, CI requirements, or approval gates.

## Available skills

### `agent-reporting`

Concise autonomous-agent completion/wait/handoff contract using `STATUS / EVIDENCE / CHANGES / VALIDATION / ROADMAP / NEXT / BLOCKER / HANDOFF`. Durable project truth stays in the target repository; the envelope carries exact verification and resume evidence.

See [`agent-reporting/README.md`](agent-reporting/README.md) and [`agent-reporting/SKILL.md`](agent-reporting/SKILL.md).

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
- Keep one skill per top-level directory.
- Use a distinctive command namespace when a skill exposes shorthand commands.
- Do not duplicate project-specific governance in shared skills.
- Treat each consuming project's submodule commit as its approved shared-skill version.
- Change shared skills in this repository, then deliberately update consuming projects to the desired commit.
