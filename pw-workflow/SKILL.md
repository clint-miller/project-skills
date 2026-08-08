---
name: pw-workflow
description: Command-driven workflow activated by the exact prefixed tokens `pw-bootstrap`, `pw-status`, `pw-review`, `pw-execute`, `pw-merge`, `pw-review-merge`, and `pw-full`. Use these commands to inspect, plan, execute, review, and conditionally merge work while reading applicable instructions, verifying authoritative current state, enforcing one work scope per chat, and preserving all authorization and approval gates. Do not treat unprefixed words such as bootstrap, status, review, execute, merge, or full as aliases.
---

# PW Workflow

Provide a project-agnostic workflow for inspecting, planning, executing, reviewing, and conditionally merging work in a structured project.

## Supported shorthand commands

### Read-only commands

- `pw-bootstrap [project_name]`
- `pw-status`
- `pw-review [pr]`

### Action commands

- `pw-execute`
- `pw-merge`
- `pw-review-merge [pr]`
- `pw-full [project_name]`

Treat these as user-intent aliases, never as permission to ignore higher-priority instructions, safety requirements, project governance, approval gates, or authorization boundaries.

Treat the `pw-` prefix as the command namespace. Activate a workflow only when the user's message starts with one of the supported `pw-*` command tokens or is an unmistakable request to use that exact PW command. Do not activate this skill merely because ordinary words such as project, bootstrap, status, review, execute, merge, or full appear in prose.

There are no legacy unprefixed command aliases. If the user sends only `bootstrap`, `status`, `review`, `execute`, `merge`, `review-merge`, or `full`, do not reinterpret it as a PW command. This prevents collisions with host projects, other skills, and ordinary conversation.

## Core principles

1. **Discover, do not assume.** Determine project structure, repositories, instructions, authoritative sources, active work, and deployment state from the current environment and available tools.
2. **Read project instructions first.** Before substantial work, locate and read applicable project/repository instructions and follow their authority order.
3. **Prefer current authoritative state.** Verify live repositories, issue trackers, PRs/MRs, CI, deployments, releases, or other authoritative systems instead of relying on stale memory or prior conversation state.
4. **Respect project governance.** Follow project-defined merge policy, deployment policy, review requirements, approval requirements, privacy/security boundaries, scope restrictions, and release rules.
5. **Preserve authorization boundaries.** A shorthand command is not blanket authority. Never widen authority beyond what the project and user permit.
6. **Do not invent unavailable source material.** If required source material cannot be retrieved, report the blocker rather than fabricating or substituting it.
7. **Keep output decision-ready.** Use exact identifiers, SHAs, versions, dates, checks, PR numbers, and deployment references when materially useful.
8. **Revalidate before irreversible actions.** Re-fetch current state immediately before merge, deployment, release, deletion, migration, or other consequential actions.

## One-project-per-chat scope lock

A chat may have only one active project scope for this workflow.

### Establish scope

- If `pw-bootstrap <project_name>` or `pw-full <project_name>` explicitly names a project, that name establishes the scope if no scope is already locked.
- If no project name is supplied, infer the project from the current chat, project/workspace context, repository, connected sources, working directory, or applicable instructions.
- Ask which project only when it genuinely cannot be resolved from available context or tools.

### Enforce scope

- Once a project is established, all later workflow commands remain within that project.
- Never silently switch project scope in the same chat.
- If the user requests work for a materially different project, stop before project work and tell them this chat is already scoped to the established project and the other project should use a new chat.
- Do not treat an issue, PR, submodule, client, component, or workstream as a different project if authoritative project instructions define it as part of the same project.
- Mentioning or comparing another project without requesting work on it does not violate the scope lock.
- The scope lock is conversation-local only; never carry it into a different chat solely from prior-chat history.

## Common bootstrap procedure

Adapt to the project and tools available. Do not assume the project uses GitHub or even software repositories.

1. **Discover instructions and authority**
   - Locate applicable project/repository instructions, README files, contribution/agent guidance, status/roadmap documents, operating policies, and relevant component instructions.
   - Follow any authority order defined by the project.

2. **Inspect authoritative live state**
   - Inspect the current baseline/default branch or authoritative equivalent.
   - Inspect active/open work: issues/tickets, PRs/MRs/change requests, milestones, backlog, releases, CI/checks, staging/production state, or analogous systems.
   - Inspect recent relevant merges/releases when needed to understand current state.

3. **Read relevant documentation**
   - Read the project-wide docs required by project instructions.
   - Read component/client/tool docs relevant to active work.
   - Inspect current version/release metadata when applicable.

4. **Reconcile state**
   - Compare live state with status, roadmap, changelog, release, deployment, and architecture documentation.
   - Flag stale, conflicting, incomplete, or unverifiable documentation instead of silently treating it as current.

5. **Assess roadmap**
   - Identify completed, active, blocked, queued, deferred, and optional work.
   - Order remaining work by dependencies, project priorities, risk, and value.
   - Respect human/manual gates and unavailable source dependencies.

6. **Check deployed/runtime behavior when relevant**
   - When deployed behavior materially affects status or the next task, verify it using available authorized means.

## Command: `pw-bootstrap [project_name]`

Read-only planning workflow.

1. Resolve and lock project scope.
2. If `project_name` was explicitly supplied, scope the entire operation to that project only.
3. Read applicable project instructions.
4. Run the common bootstrap procedure.
5. Provide:
   - **Project status** — current authoritative baseline and meaningful active state;
   - **Roadmap** — ordered remaining work, blockers, approvals, and deferred/future items where relevant;
   - **Recommended next task** — one concrete highest-value safe next task;
   - **Next task prompt** — a complete prompt for ChatGPT to execute that task later.
6. The task prompt should carry forward verified identifiers, startup checks, scope, objective, known current state, validation expectations, authorization boundaries, stop conditions, and source dependencies.
7. Do not modify project state during `pw-bootstrap`.

## Command: `pw-status`

Read-only status workflow.

1. Resolve or enforce the chat's locked project scope.
2. Re-read applicable project instructions as necessary.
3. Re-run enough of the common bootstrap procedure to establish current authoritative state.
4. Provide:
   - **Project status**;
   - **Roadmap**.
5. Include blockers, risks, and stale/conflicting documentation when relevant.
6. Do not generate a next-task prompt unless asked.
7. Do not modify project state.

Do not interpret ordinary non-project uses of the word `pw-status` as this workflow command.

## Command: `pw-review [pr]`

Review the identified PR/MR and provide a merge recommendation.

1. Enforce the locked project scope.
2. Resolve the target PR/MR. `pr` may be a number, URL, branch, or other unambiguous project-native identifier.
3. Run the common PR/MR review procedure.
4. Do not merge or modify project state.
5. Output the relevant PR/MR identifiers, exact head/base revisions where material, check state, important findings, blockers/follow-up, and explicit merge recommendation.

If no PR/MR identifier is supplied, use the clearly established target from immediately preceding project context only when unambiguous. Otherwise ask for the target rather than guessing.

## Common PR/MR review procedure

Use for `pw-review` and `pw-review-merge`, and when `pw-full` produces or encounters a directly relevant PR/MR.

1. Identify the target PR/MR from the command, immediately preceding context, or the directly relevant work created by the workflow. Do not guess among multiple plausible targets.
2. Read applicable project instructions and PR/MR-specific contribution/review requirements.
3. Fetch current live PR/MR state including, when available:
   - identifier/title;
   - base and head branch/revision;
   - draft/open/closed/merged state;
   - mergeability/conflicts;
   - changed files and diff;
   - commits ahead/behind;
   - required and recent checks/CI;
   - review discussions/approvals;
   - linked issue/ticket;
   - preview/staging/deployment evidence;
   - version/changelog/release/registry documentation required by the project.
4. Perform a skeptical review for correctness, regressions, scope creep, missing tests, stale docs, security/privacy/architecture effects, release/version consistency, and unmet project gates.
5. Distinguish merge blockers from non-blocking follow-up.
6. Give an explicit recommendation:
   - **Recommend merge**;
   - **Do not recommend merge**; or
   - **Recommend merge after specified conditions**.
7. A review alone is read-only.

## Command: `pw-execute`

Execute the recommended task prompt from the immediately previous assistant reply.

1. Enforce the locked project scope.
2. Identify the recommended task prompt from the immediately previous assistant reply.
3. If there is no clear immediately preceding task prompt, do not guess. State that there is no preceding recommended task prompt to execute and direct the user to `pw-bootstrap` or to specify the task.
4. Treat that prompt as the user's intended task, not as immutable current-state facts.
5. Before changes:
   - read applicable project instructions;
   - revalidate authoritative current state;
   - verify relevant identifiers, branch/PR/MR state, SHAs, checks, deployments, and dependencies;
   - detect whether the recommendation is stale, already completed, unsafe, blocked, or superseded.
6. Safely adjust stale mutable details only when doing so preserves the same task, scope, and authorization boundaries.
7. Execute as much as can safely be completed in the current turn with available tools.
8. `pw-execute` itself does not grant new merge, deployment, destructive-change, infrastructure, spending, privacy, security, or release authority.
9. Report what was done, validation performed, resulting authoritative state, what was intentionally not done, and the next meaningful action if work remains.

## Command: `pw-merge`

Perform the merge that was explicitly recommended in the immediately previous assistant reply.

1. Enforce the locked project scope.
2. Identify exactly one merge recommendation from the immediately previous assistant reply.
3. If the previous reply did not clearly recommend merging a single PR/MR, do not guess and do not merge.
4. Treat the user's exact `pw-merge` command as authorization to perform that previously recommended merge **only to the extent project governance permits that authorization model**.
5. Before merging:
   - re-read applicable project instructions;
   - re-fetch PR/MR and current base state;
   - confirm the target and, when recommendation was exact-head, confirm head revision is unchanged;
   - verify mergeability, required checks, reviews, preview/staging evidence, manual gates, release/version/docs requirements, and other project-specific prerequisites.
6. If material state changed, stop and report it unless a fresh same-turn review still supports the exact authorized merge under project rules.
7. Use the project's required merge method. Never force-push or rewrite shared history to make a merge possible.
8. Verify resulting merge state and required post-merge checks/deployment.
9. Report, when available:
   - PR/MR identifier/title;
   - reviewed/merged head revision;
   - base revision at merge time;
   - relevant checks;
   - merge method;
   - resulting merge/squash commit;
   - linked issue/ticket state;
   - deployment/release result;
   - remaining follow-up.

## Command: `pw-review-merge [pr]`

Freshly review the target PR/MR and merge it in the same turn if recommended and authorized.

1. Enforce the locked project scope.
2. Resolve the target PR/MR.
3. Run the common PR/MR review procedure against current state.
4. If the result is **Recommend merge**, proceed to merge only if all project-specific approval and merge gates are satisfied.
5. If project governance requires separate approval after review, or does not permit advance conditional merge authorization, stop after review and report that required gate.
6. If the review does not recommend merge, do not merge.
7. Treat the user's exact `pw-review-merge` command as conditional authorization for the directly identified PR/MR only; it grants no authority over unrelated work.
8. Immediately before merging, revalidate current state and use the same merge/output requirements as `pw-merge`.
9. Output relevant PR review evidence and, if merged, resulting merge information.

## Command: `pw-full [project_name]`

End-to-end project workflow: instructions → bootstrap → recommendation → execution → applicable PR/MR review → conditional merge → final evidence.

1. Resolve and lock project scope. If `project_name` is explicitly supplied, scope the entire operation to that project only.
2. Read applicable project instructions and authority rules.
3. Run the common bootstrap procedure.
4. Establish current **Project status** and **Roadmap** internally and report meaningful findings as work progresses when appropriate.
5. Select the single highest-value safe recommended next task.
6. Construct the complete ChatGPT task prompt that `pw-bootstrap` would have returned.
7. Execute that recommended task in the same turn, after revalidating live mutable state.
8. If execution creates, updates, or directly depends on a single relevant PR/MR:
   - perform the common PR/MR review procedure;
   - if the review recommends merge and project governance permits the user's advance conditional authorization, merge it;
   - otherwise stop at the applicable approval/gate without bypassing it.
9. Do not broaden into unrelated work merely because adjacent issues/PRs are visible.
10. Report final authoritative evidence, including as applicable:
    - starting baseline;
    - selected task and what was executed;
    - files/issues/branches/PRs affected;
    - validation and CI/checks;
    - PR/MR review findings and recommendation;
    - exact reviewed/merged revisions;
    - merge method and resulting commit;
    - linked issue/ticket result;
    - deployment/release result;
    - blockers, residual risks, or next action.

The exact `pw-full` command is advance conditional authorization to carry the recommended task through its directly applicable review/merge stage only where project governance allows that model. It never overrides a project rule requiring later explicit approval, manual acceptance, security review, production activation approval, or other non-delegable gate.

## Output quality checks

Before finishing any command:

- Confirm the response stayed inside the locked project scope.
- Confirm applicable project instructions were consulted as required.
- Confirm authoritative current state was checked when available.
- Distinguish verified facts from assumptions or unavailable evidence.
- Flag stale/conflicting documentation.
- Preserve explicit approval and authorization boundaries.
- Avoid proposing or executing work already complete.
- Revalidate before irreversible actions.
- Report exact identifiers and evidence when useful for future handoff.
