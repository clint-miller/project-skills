# Repository Agent Instructions

This repository is the canonical home for reusable, project-agnostic skills shared across managed project repositories.

## Bootstrap order

For fresh work in this repository:

1. Read this `AGENTS.md`.
2. Read `.director/project.yaml` for repository identity and onboarding/authority boundaries.
3. Read the root `README.md` for the human-facing repository overview and current skill inventory.
4. Read the relevant `<skill-name>/SKILL.md` before changing or evaluating a skill. A skill-local `README.md` is explanatory documentation; `SKILL.md` is the authoritative skill definition.

The authoritative repository identity is `clint-miller-projects/project-skills`. Historical `clint-miller/project-skills` references that may still appear in human-facing documentation are stale location references, not repository authority, and should be corrected only through explicit Project Skills work.

## Scope and governance

- Keep shared skills project-agnostic and composable.
- Project-specific governance, repository instructions, safety rules, release policy, CI requirements, and approval gates outrank shared skill guidance.
- Director 4 runtime lifecycle identity, heartbeat, routing, recovery, and authoritative-return semantics remain Director-owned baseline behavior; shared skills must not become required for D4 execution correctness.
- Changes in this repository do not automatically change consuming projects. Consumer submodule/reference/pin updates, deployment, or activation are separate explicitly authorized adoption work.
- Do not update a consuming repository's pin/reference while implementing or reviewing a Project Skills change unless a separate task explicitly authorizes that adoption.

## Change and review workflow

Use a bounded branch and pull request for repository changes. Review the exact diff against the relevant `SKILL.md`, keep any skill-local `README.md` consistent with the authoritative definition, and verify that repository identity/governance boundaries remain accurate.

At onboarding time this repository has no repository-native CI workflow. Until a later explicit task adds one, verification is evidence-backed review of the exact branch/PR diff plus any deterministic checks introduced by the change itself. Do not invent a passing CI claim.

## Director 4 lifecycle

When Director 4 dispatches work to this native Project, use the execution-specific lifecycle contract supplied in the task and the permanent Project Skills status mailbox registered in Director. Browser-visible ChatGPT completion is not authoritative lifecycle completion evidence.
