# Autonomous Development Cycle

**Applies to:** All repositories

## 8-Step Cycle

1. **Analyze** - Read issue, understand requirements, identify root cause
2. **Branch** - Create feature branch: `{type}/issue-{num}-{description}`
3. **Fix** - Implement solution with minimal changes
4. **Test** - Write and run comprehensive tests
5. **Lint** - Fix ALL pre-commit hook issues (see [lint-and-hooks.md](lint-and-hooks.md))
6. **PR** - Create pull request with detailed description (see [pr-guidelines.md](pr-guidelines.md))
7. **Merge** - Squash and merge when tests pass
8. **Cleanup** - Delete branch, verify clean tree

## Critical Principles

❌ **NEVER** use `--no-verify` in autonomous mode
❌ **NEVER** bypass pre-commit hooks
❌ **NEVER** wait for user review (autonomous mode)
✅ **ALWAYS** fix hook failures by fixing code
✅ **ALWAYS** read step-specific guides before executing
✅ **ALWAYS** complete entire cycle without user intervention

## Step-Specific Guides

**Read just-in-time:**
- **Step 5 (Lint):** [lint-and-hooks.md](lint-and-hooks.md) - Before first commit
- **Step 6 (PR):** [pr-guidelines.md](pr-guidelines.md) - Before creating PR

## Project-Specific Details

Check for `docs/CLAUDE.md` or `docs/guides/` in current repo for project-specific workflow details.
