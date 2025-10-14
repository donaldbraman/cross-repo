# General Claude Code Instructions

**Applies to:** All repositories

## Autonomous Development Cycle

8-step cycle: Analyze → Branch → Fix → Test → Lint → PR → Merge → Cleanup

**Full details:** [~/.claude/guides/autonomous-cycle.md](file:///Users/donaldbraman/.claude/guides/autonomous-cycle.md)

## Critical Rules

- ❌ NEVER use `--no-verify` in autonomous mode
- ❌ NEVER bypass pre-commit hooks
- ✅ ALWAYS fix hook failures by fixing code
- ✅ ALWAYS read step guides before executing:
  - Step 5 (Lint): `~/.claude/guides/lint-and-hooks.md`
  - Step 6 (PR): `~/.claude/guides/pr-guidelines.md`

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

## Project-Specific Instructions

Each repository has project-specific instructions:
1. Check for `docs/CLAUDE.md` in current repo
2. Check for `docs/guides/` directory
3. Follow project-specific workflow details

---

*Last updated: 2025-10-14*
