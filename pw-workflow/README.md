# PW Workflow Skill v0.4.0

A project-agnostic, command-driven ChatGPT Skill using a distinctive `pw-` namespace to avoid collisions with project names, host workspaces, other skills, and ordinary conversation.

## Commands

### Read-only

- `pw-bootstrap [project_name]`
- `pw-status`
- `pw-review [pr]`

### Actions

- `pw-execute`
- `pw-merge`
- `pw-review-merge [pr]`
- `pw-full [project_name]`

There are **no unprefixed aliases**. Bare words such as `bootstrap`, `status`, `review`, `execute`, `merge`, or `full` are intentionally not commands for this skill.

The skill preserves the existing one-project-per-chat scope lock, revalidates authoritative live state before actions, and honors project-specific authorization and approval gates.
