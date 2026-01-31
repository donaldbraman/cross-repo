# General Claude Code Instructions

**Applies to:** All repositories

## Autonomous Development Cycle

Standard Dev Cycle: Analyze & Research → GitHub Issue & Worktree → Dev & Test → Lint → Commit, Push & Merge → Close Issue & Delete Worktree

**Details:** [~/.claude/guides/autonomous-cycle.md](file:///Users/donaldbraman/.claude/guides/autonomous-cycle.md)

## Critical Rules

- ❌ NEVER use `--no-verify` in autonomous mode
- ❌ NEVER bypass pre-commit hooks
- ❌ NEVER work in main repo directory (use worktrees)
- ✅ ALWAYS fix hook failures by fixing code
- ✅ ALWAYS use worktrees for isolated development
- ✅ ALWAYS read step guides before executing:
  - Step 4 (Lint): `~/.claude/guides/lint-and-hooks.md`
  - Step 5 (PR): `~/.claude/guides/pr-guidelines.md`

## GitHub Labels

**Always auto-generate** when creating issues/PRs. Enables historical search.

**Details:** [~/.claude/guides/github-labels.md](file:///Users/donaldbraman/.claude/guides/github-labels.md)

## Agent-to-Agent Messaging

**Protocol for AI agents communicating across repositories.**

**Details:** [~/.claude/guides/agent-messaging.md](file:///Users/donaldbraman/.claude/guides/agent-messaging.md)

## Git Workflow

**Standard git practices across all projects.**

**Details:** [~/.claude/guides/git-workflow.md](file:///Users/donaldbraman/.claude/guides/git-workflow.md)

## Code Versioning

**Semantic versioning and version management patterns.**

**Details:** [~/.claude/guides/code-versioning.md](file:///Users/donaldbraman/.claude/guides/code-versioning.md)

## Sibling Repository Map

**Local repos and their relationships.**

**Details:** [~/.claude/guides/sibling-repos.md](file:///Users/donaldbraman/.claude/guides/sibling-repos.md)

## Project-Specific Instructions

Each repository has project-specific instructions:

1. Check for `docs/CLAUDE.md` in current repo
2. Check for `docs/guides/` directory
3. Follow project-specific workflow details

---

*Last updated: 2026-01-31*
