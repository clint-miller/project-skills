---
name: director-workflow
description: Director control-plane workflow for queue status, project status, adding explicit or derived work, planning next work from repository truth, queue control, and reconciliation. Use when the user asks what Director or a managed project is doing, what needs attention, to add or prioritize work, to determine work from a repository's own status/roadmap, or to reconcile Director state with repository/CI/deployment evidence. Enforces durable repository documentation synchronization before successful handoff.
---

# Director Workflow

Provide a consistent human-to-Director control contract across managed project repositories.

Director coordinates work. The managed repository remains authoritative for project-specific status, roadmap, decisions, constraints, and resumable technical state. Git/PR/CI/deployment systems provide proof of work. Director's queue/runtime state is authoritative only for orchestration state.

## Supported operations

The following explicit commands are preferred shorthand:

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

Natural-language requests with the same unmistakable intent may invoke the corresponding operation. Examples:

- `What is the Director status?` -> `director-status`
- `What is going on with Futures?` -> `director-project-status Futures`
- `Add this to Futures: repair the protected staging acceptance check.` -> explicit `director-add-work`
- `Look at Futures and add whatever it actually needs next.` -> derived `director-add-work`
- `Figure out the next Atlas work from its repo status.` -> `director-plan-work Atlas`
- `Make Futures the top priority.` -> `director-control`
- `Reconcile Director with reality.` -> `director-reconcile`
- `What needs me?` -> `director-needs-me`

Do not require the user to memorize exact command strings when intent is otherwise unambiguous.

## Authority model

Use these authority layers and do not collapse them:

1. **Managed repository = project truth**
   - project status;
   - roadmap and next steps;
   - durable decisions and locked paths;
   - architecture and project-specific instructions;
   - blockers and resumable technical state.

2. **Director = orchestration truth**
   - queue order;
   - current assignment/ownership;
   - execution lifecycle state;
   - handoff/wait relationships;
   - control-plane checkpoints.

3. **Git/issue/PR/CI/deployment systems = proof of work**
   - commits and exact SHAs;
   - issues/PRs;
   - workflow runs/checks;
   - deployments/releases/runtime acceptance.

4. **Conversation = user intent, not durable project memory**
   - use current user instructions as authority for requested changes;
   - do not treat old chat narrative as stronger than current durable state.

When these disagree, identify the disagreement and reconcile it rather than silently choosing whichever source is convenient.

## Core principles

1. **Discover, do not assume.** Resolve managed repositories, current state, applicable instructions, and Director state from authoritative sources.
2. **Read repository guidance before deciding project work.** Status and planning operations must inspect applicable project instructions and status/roadmap documentation.
3. **Prove activity.** Distinguish scheduler activity, code churn, or repeated CI runs from meaningful progress.
4. **Prefer exact evidence.** Use commit SHAs, PR/issue numbers, CI run IDs, deployment identifiers, timestamps, and durable paths when useful.
5. **Do not invent blockers or approvals.** Human action is required only when project governance or current evidence genuinely requires it.
6. **Do not disrupt productive active work by default.** Adding work normally queues it. Reprioritize or preempt only when the user explicitly requests it or project governance requires it.
7. **Keep project docs current.** Material project state changes must be reflected durably in the managed repository before successful handoff.
8. **A fresh agent must be able to resume from durable state.** Never make chat history necessary for safe continuation.

# Issue-backed Director intake and planning contract

Use this contract when a managed repository participates in the accepted Director-managed GitHub Issue lifecycle. It is an intake/planning contract only; lifecycle scanning and label mutation remain separate implementation concerns unless another authorized workflow explicitly provides them.

## Lifecycle semantics

The accepted v1 lifecycle is:

```text
no Director label
-> director:approved
-> director:queued
-> director:active
-> director:verify
-> director:done
-> closed
```

`director:blocked` is the temporary blocked state from queued/active work and may return to the appropriate actionable state after reconciliation.

`director:approved` means **the owner authorized Director intake**, not that code execution is automatically authorized. Approval requires Director to inspect, normalize, deduplicate, and determine whether planning/research is required before an executable Repo assignment exists.

Director runtime remains orchestration authority. The GitHub issue remains the project-local request, rationale, discussion, and narrative history. Do not copy the entire issue body or mutable label state into Director runtime as a second authoritative record.

## Immutable issue identity and deduplication

Treat the tuple below as the stable identity of issue-backed work:

```text
origin.repository + origin.issue_number
```

Before creating or adding Director work from an approved issue:

1. search active/non-terminal Director work for that exact repository + issue number identity;
2. if linked work already exists, reconcile/update that work instead of creating a duplicate;
3. do not deduplicate solely by title, issue text, branch name, or conversational similarity;
4. terminal historical work remains historical; a reopened/renewed request may require a successor work lineage rather than mutation of the completed record.

Repeated intake of the same approved issue must therefore be idempotent.

## Normalizing an approved issue into Director work

When Director accepts approved issue-backed work, create or update the smallest linked orchestration record needed for autonomous continuation. Preserve the issue as narrative authority and normalize only execution-relevant facts, including when applicable:

- immutable issue repository + issue number reference;
- normalized objective and acceptance criteria;
- priority/dependencies owned by Director;
- current planning requirement/status;
- structured plan task/group/checkpoint identifiers when planning is required;
- exact next ready task(s), only after dependencies/checkpoints are satisfied;
- stop/block/human-required conditions;
- repository/branch/fence/context details only when an executable bounded assignment is actually materialized.

Do not treat issue approval itself as permission to assign a Repo Agent. An executable assignment still requires the normal Director dispatch contract, repository readiness, scope, context packet, fence/branch identity, and capacity checks.

## Planning requirement

Require structured research/planning/decomposition when **any** of these conditions apply:

- architecture or control-plane change;
- cross-repository work;
- unknown root cause or substantial discovery;
- more than one dependent executable task;
- schema, protocol, or governance change;
- high-risk or difficult-to-reverse change;
- deployment/production implications;
- acceptance requires multiple independent checkpoints.

A direct bounded dispatch is allowed only when **all** of these conditions are true:

- objective is already clear;
- one small independent task can complete it;
- risk is low and reversible;
- acceptance criteria are obvious and verifiable;
- no unresolved dependency or research question remains.

For planned work, persist the dispatch graph/status durably enough that a fresh Director can determine which task is ready next. Narrative research may stay in issue comments or project artifacts, but executable dependency/checkpoint state must not depend on chat memory.

If new repository evidence invalidates the current plan, revise the durable plan and record the reason rather than silently improvising around dependency or checkpoint rules.

## Operation integration

Apply the issue-backed contract consistently to existing Director operations:

- `director-status` / `director-project-status`: show linked issue identity and reconciled lifecycle when issue-backed work is materially active; do not imply `director:approved` means implementation is active.
- `director-add-work`: when the request originates from or resolves to an approved issue, deduplicate by immutable issue identity before creating linked work.
- `director-add-work --derive`: approved issue intake is evidence for authorized work-management intake, but derived executable work still requires repository truth, planning/dependency checks, and normal dispatch authority.
- `director-plan-work`: apply the planning criteria above and persist structured task/dependency/checkpoint state when non-trivial work requires it.
- `director-reconcile`: detect duplicate linked work, issue/runtime mismatch, material issue edits, manual closure, or stale assumptions; fail closed on ambiguous owner intent rather than continuing execution blindly.

This section does **not** authorize registered-repository issue scanning, lifecycle label writes, issue closure, Repo Agent issue-progress comments, dashboard mutation, scheduler changes, or managed-project rollout. Those behaviors require their separately authorized implementation phases.

# Cross-repo Director submission from a managed project chat

A user may invoke Director control-plane operations while working inside a specific managed project/repository chat.

This is a first-class workflow and does **not** by itself switch the chat's project work scope.

When the current chat is clearly scoped to a managed project:

- `Add this to Director: <request>` means add work for the current project unless the user names another target.
- `Queue this for later: <request>` may be treated as explicit `director-add-work` for the current project when Director intent is unmistakable from context.
- `Add whatever this project needs next to Director` means derived `director-add-work` for the current project.
- `What does Director have queued for this project?` means `director-project-status` for the current project.

The current managed project may therefore remain the working scope while the skill writes the minimum required orchestration mutation to Director's authoritative control-plane state.

This control-plane submission is not implementation work in the Director repository. Do not begin unrelated Director development, refactoring, or project execution merely because Director state must be updated.

When `pw-workflow` has locked the chat to the current project, a Director queue submission **for that same project** is compatible with the scope lock because it is orchestration of the current project, not a switch to a second project.

If the user explicitly names a different managed project, a narrowly bounded Director queue/control mutation may still be recorded when the target and requested work are already unambiguous. Do not use that as a pretext to perform deep implementation or derived planning in the other project when doing so would violate the current chat's project-work scope.

If the Director repository/control plane is not writable from the current environment, persist the request through any project-native Director intake/handoff mechanism defined by the architecture. If no authorized durable path is available, report the blocker rather than pretending the work was queued.

# Repository documentation synchronization invariant

Repository documentation freshness is a Director lifecycle requirement, not optional maintenance.

For every material project work cycle, use this lifecycle:

```text
Inspect -> Work -> Verify -> Synchronize repo state -> Handoff
```

Before a work cycle may be reported as successfully handed off, completed, returned to Director, or ready for the next independent agent:

1. Determine which repository-native documents or state files are authoritative for current status, roadmap, decisions, blockers, and next steps.
2. Update those durable sources when the completed work materially changed any of them.
3. Record enough exact evidence for another agent to verify the result.
4. Confirm obsolete next steps are removed or marked complete so they cannot be re-executed as current work.
5. Confirm newly discovered blockers, waits, decisions, and locked paths are durable when material to continuation.
6. Only then perform the handoff/reporting step.

If repository instructions prohibit or prevent the needed documentation/state update, do not falsely report a clean successful handoff. Report the exact unsynchronized state and blocker/owner required to resolve it.

Do not create documentation churn for immaterial changes. The invariant applies when project truth actually changed.

When `agent-reporting` is also available, use its reporting envelope after this synchronization requirement is satisfied. The report summarizes durable truth; it does not replace synchronization.

# Common project inspection procedure

Use this procedure for project status, derived work, planning, and reconciliation. Adapt to available tools and the repository's own governance.

1. **Resolve the managed project and repository**
   - Prefer Director's durable project registry/mapping when one exists.
   - Otherwise resolve from current authoritative context.
   - Do not guess between multiple plausible repositories.

2. **Read applicable repository instructions**
   - agent/project instructions;
   - README/contribution guidance;
   - status/roadmap/next-step docs;
   - decision/architecture records relevant to current work;
   - project-native Director/checkpoint state when present.

3. **Inspect live proof of work**
   - recent commits and current branch/head where relevant;
   - active/recent issues and PRs;
   - CI/check state and exact candidate SHA;
   - deployment/runtime state when it materially affects project status;
   - recent failures/regressions when they bear on claimed progress.

4. **Inspect Director orchestration state**
   - queued/running/waiting/blocked/completed lifecycle;
   - assigned worker/agent;
   - expected next handoff;
   - last meaningful Director/repo check-in;
   - priority and dependencies.

5. **Reconcile the sources**
   - compare repository docs, Director state, and proof of work;
   - identify stale, contradictory, duplicated, or already-completed work;
   - distinguish real progress from repeated attempts without net movement.

6. **Determine the next durable state**
   - status only: report it without mutation;
   - planning: propose justified work without mutation unless the requested operation includes adding work;
   - reconcile: repair stale durable state within available authority;
   - add-work: create normalized durable work items only after deduplication and dependency checks.

# Operation: `director-status`

Provide a read-only status of the entire Director-managed queue.

Inspect enough authoritative state to avoid merely repeating stale Director metadata.

Report projects grouped or clearly labeled by lifecycle, using the project's native lifecycle terms when defined. At minimum distinguish:

- running/active;
- waiting on CI or another normal asynchronous dependency;
- blocked;
- queued/ready;
- idle/complete.

For each materially active project include, when available:

- project name;
- current objective/work item;
- current owner/worker;
- last meaningful proof of work;
- current wait/blocker;
- next intended action.

Then include:

**Needs Clint:** only actions that genuinely require the user's decision, authorization, credential, manual acceptance, or unavailable source material. If none, say `None`.

Do not list ordinary CI waiting, agent scheduling, or routine autonomous next steps as user-required work.

# Operation: `director-project-status <project>`

Provide a read-only, evidence-based status for one managed project.

Run the common project inspection procedure deeply enough to answer:

- **Current objective** — what the project is actually trying to accomplish now;
- **Actual progress** — meaningful completed movement, not raw activity;
- **Proof** — exact recent evidence supporting the progress claim;
- **Current work** — what is presently assigned/running/waiting;
- **Last completed step** — last verified meaningful milestone;
- **Blocker/wait state** — exact unresolved dependency, if any;
- **Roadmap** — remaining ordered work from repository truth, reconciled with live evidence;
- **Next step** — one justified next action;
- **Needs Clint** — only if human action is genuinely required.

Explicitly call out suspected loops/regression churn when recent activity repeatedly revisits the same failure without durable forward movement.

Do not mutate project or Director state during status unless the user also requested reconciliation or another write operation.

# Operation: `director-add-work <project>: <request>`

Add explicit user-requested work to the managed project.

1. Resolve project/repository and read applicable guidance.
2. Inspect current roadmap/queue enough to detect duplicate, already-completed, contradictory, or superseded work.
3. Preserve the user's requested outcome; normalize implementation wording only as needed for a fresh agent to execute it safely.
4. Record the work in the project's approved durable work/roadmap system and/or Director queue according to the architecture.
5. Preserve dependencies, locked decisions, and approval gates.
6. Queue by default. Do not interrupt productive active work unless the user explicitly requests priority/preemption or the project defines a mandatory safety/correctness preemption.
7. Synchronize any repository docs whose project truth changed because the work was added or reprioritized.
8. Report exactly what was added, where it was persisted, its priority/dependencies, and whether it affected current execution.

## Derived mode: `director-add-work <project> --derive`

Use when the user wants Director to determine and add needed work instead of supplying the task.

This mode MUST NOT invent backlog from generic best practices. Derive work from the project's own authoritative state and live evidence.

1. Run the common project inspection procedure with `pw-status`-like depth.
2. Reconcile repository status/roadmap with commits, issues/PRs, CI, deployment/runtime evidence, and Director state.
3. Identify work that is:
   - explicitly remaining in repository truth;
   - required to resolve a verified blocker/failure;
   - required to complete an already-authorized objective;
   - required to repair stale/conflicting durable state;
   - a justified next dependency of the current roadmap.
4. Exclude:
   - already-completed work;
   - speculative enhancements not supported by project intent;
   - abandoned/superseded paths;
   - work prohibited by locked decisions;
   - duplicates of current/queued work.
5. Normalize the justified tasks into executable durable work items.
6. Order them by dependency, stated project priority, risk, and value.
7. Persist the selected work through the approved project/Director mechanism.
8. Update repository status/roadmap docs if the reconciliation changes project truth.
9. Report the evidence used to derive the work and exactly what was added.

If evidence does not justify new work, add nothing and say so.

# Operation: `director-plan-work <project>`

Determine what the project should work on next without automatically adding it unless the user's wording explicitly requests queuing.

Run the same derived-work analysis as `director-add-work --derive`, then provide:

- reconciled current project state;
- ordered justified candidate tasks;
- one recommended next task;
- dependencies/blockers;
- evidence supporting the recommendation;
- any stale repo documentation that should be repaired.

This is the Director equivalent of using the repository's status/roadmap plus live evidence to decide the next work slice.

# Operation: `director-control <instruction>`

Apply an explicit orchestration change such as prioritizing, pausing, resuming, cancelling, superseding, or moving work.

1. Resolve the exact target(s).
2. Re-read current Director/project state immediately before mutation.
3. Preserve project governance and dependency constraints.
4. Avoid destructive cancellation of productive work when a non-destructive queue change satisfies the request.
5. Persist the orchestration change in Director's authoritative state.
6. Synchronize managed repository state if project truth or resumable ownership materially changed.
7. Verify the resulting state.
8. Report the before/after effect and any work intentionally left running.

A general request to add work is not implicit authority to cancel or preempt unrelated work.

# Operation: `director-reconcile`

Reconcile the entire Director queue against managed repository truth and live proof of work.

For each active, waiting, blocked, or recently completed project:

1. Inspect Director state.
2. Inspect repository-native status/roadmap/checkpoint state.
3. Inspect relevant commits/PRs/issues/CI/deployments.
4. Detect drift such as:
   - completed work still marked active;
   - queued work already completed elsewhere;
   - dead/stale ownership;
   - waiting CI whose run is already terminal;
   - stale blocker state;
   - duplicate work items;
   - roadmap/status docs lagging actual verified work;
   - Director queue claims unsupported by repository evidence.
5. Repair durable Director state within current authority.
6. Repair managed repository documentation/state when current project truth is stale and the change is unambiguous and authorized.
7. Do not rewrite ambiguous project intent merely to make systems agree; surface genuine conflicts instead.
8. Report repaired drift plus unresolved conflicts and genuine human-required items.

# Operation: `director-reconcile <project>`

Perform the same reconciliation for one managed project with deeper project-specific inspection.

A project-level reconcile should leave these mutually consistent to the extent current authority and evidence allow:

```text
Repository project truth <-> Director orchestration truth <-> live proof of work
```

# Operation: `director-needs-me`

Return only unresolved items across managed projects that genuinely require the user.

Include the project, exact decision/action required, why autonomous progress cannot continue without it, and the consequence of waiting.

Do not include:

- normal CI/deployment waits;
- work another agent can perform;
- optional preferences that do not block progress;
- routine queue management owned by Director;
- approvals that project governance does not actually require.

If nothing requires the user, return `None`.

# Meaningful progress test

When reporting progress, use this test:

A project has meaningful progress when authoritative evidence shows that a blocker was removed, a required capability was added/fixed, a verified milestone advanced, a required validation passed, project truth was materially reconciled, or the project moved to a legitimately later lifecycle state.

The following alone are not sufficient evidence of progress:

- a worker ran;
- a task was enabled/rescheduled;
- a branch was created;
- commits were produced without validation or objective advancement;
- CI ran repeatedly on equivalent failing candidates;
- the same defect was rediscovered without a durable fix;
- status text changed without corresponding project evidence.

When uncertain, say that activity is visible but meaningful progress is not yet proven.

# Work-item normalization contract

When creating Director-managed work, persist enough information for a fresh agent to act without chat history. Include, when applicable:

- project/repository;
- concise objective;
- source/reason for the work;
- priority;
- dependencies;
- relevant locked decisions/constraints;
- current known evidence/identifiers;
- required validation/acceptance;
- stop/block conditions;
- expected durable state/docs to update before handoff.

Do not embed broad coding methodology, self-learning theory, or unrelated historical narrative in each work item. Project instructions and shared skills should carry reusable process rules.

# Output discipline

Keep Director responses decision-ready.

- Lead with actual state or action taken.
- Prefer concise evidence over narrative.
- Clearly distinguish `active`, `waiting`, `blocked`, `queued`, and `complete`.
- Clearly distinguish **activity** from **meaningful progress**.
- Put genuine user-required items under **Needs Clint**.
- Do not claim external effects that were not verified.
- Do not claim project completion merely because one bounded task completed.
- Do not expose private chain-of-thought.
