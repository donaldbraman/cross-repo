# Autonomous Development Cycle

**Version:** 2.0.0
**Last Updated:** 2025-01-19
**Applies to:** All repositories

## 6-Step Cycle

1. **Analyze & Research** - Read issue, understand requirements, explore codebase
2. **Issue & Worktree** - Create/update detailed GitHub issue, create linked worktree
3. **Develop & Test** - Implement solution, write tests, verify everything works
4. **Lint** - Fix ALL pre-commit hook issues (see [lint-and-hooks.md](lint-and-hooks.md))
5. **Commit, Push & Merge** - Create PR, squash merge (see [pr-guidelines.md](pr-guidelines.md))
6. **Close & Cleanup** - Close issue, delete worktree

## Quick Reference

```bash
# === STEP 2: Issue & Worktree ===
gh issue create --title "feat: Description" --label "enhancement" --body "..."
cd ~/Documents/GitHub/{repo}
git checkout main && git pull origin main
mkdir -p ../{repo}-worktrees
git worktree add ../{repo}-worktrees/issue-{N} -b feat/{N}-description
cd ../{repo}-worktrees/issue-{N}

# === STEP 3-4: Develop & Test & Lint ===
# ... make changes, run tests, fix lint issues ...
git add . && git commit -m "feat: ..."

# === STEP 5: Commit, Push & Merge ===
git push -u origin feat/{N}-description
gh pr create --title "feat: Description" --body "... Closes #{N}"
gh pr merge --squash --delete-branch

# === STEP 6: Close & Cleanup ===
cd ~/Documents/GitHub/{repo}
git worktree remove ../{repo}-worktrees/issue-{N}
git pull origin main
git worktree prune
```

## Critical Principles

❌ **NEVER** use `--no-verify` in autonomous mode
❌ **NEVER** bypass pre-commit hooks
❌ **NEVER** work in main repo directory (use worktrees)
✅ **ALWAYS** fix hook failures by fixing code
✅ **ALWAYS** use worktrees for isolated development
✅ **ALWAYS** complete entire cycle without user intervention

## Step Details

### Step 1: Analyze & Research

- Read the issue or user request thoroughly
- Explore relevant code to understand current implementation
- Identify root cause (for bugs) or integration points (for features)
- Note any dependencies or related files

### Step 2: Issue & Worktree

**Create/update GitHub issue** with detailed description:
- Problem statement or feature request
- Acceptance criteria
- Technical approach (if known)

**Create worktree** linked to issue:
```bash
git worktree add ../{repo}-worktrees/issue-{N} -b feat/{N}-description
cd ../{repo}-worktrees/issue-{N}
```

### Step 3: Develop & Test

Work entirely within the worktree directory:
- Implement minimal changes to solve the problem
- Write or update tests
- Run tests until all pass
- Verify functionality works as expected

### Step 4: Lint

Before committing, ensure all hooks pass:
- Run `git add . && git commit`
- If hooks fail, fix the code (never `--no-verify`)
- See [lint-and-hooks.md](lint-and-hooks.md) for common fixes

### Step 5: Commit, Push & Merge

```bash
git push -u origin feat/{N}-description
gh pr create --title "feat: Description" --body "Closes #{N}"
gh pr merge --squash --delete-branch
```

See [pr-guidelines.md](pr-guidelines.md) for PR format.

### Step 6: Close & Cleanup

```bash
cd ~/Documents/GitHub/{repo}
git worktree remove ../{repo}-worktrees/issue-{N}
git pull origin main
git worktree prune
```

Issue closes automatically if PR body contains "Closes #{N}".

## Step-Specific Guides

**Read just-in-time:**
- **Step 2 (Worktree):** [worktree-dev-cycle.md](worktree-dev-cycle.md) - Worktree setup and management
- **Step 4 (Lint):** [lint-and-hooks.md](lint-and-hooks.md) - Pre-commit hook handling
- **Step 5 (PR):** [pr-guidelines.md](pr-guidelines.md) - PR creation standards

## Related Guides

- [Worktree Dev Cycle](worktree-dev-cycle.md) - Detailed worktree workflow
- [Lint & Hooks](lint-and-hooks.md) - Pre-commit hook handling
- [PR Guidelines](pr-guidelines.md) - Pull request creation
- [Git Workflow](git-workflow.md) - Branch and commit conventions
- [GitHub Labels](github-labels.md) - Label creation for issues/PRs

---

*Last updated: 2025-01-19*
