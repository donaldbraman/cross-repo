# Autonomous Development Cycle

**Version:** 2.1.1
**Last Updated:** 2025-01-20
**Applies to:** All repositories

## 6-Step Cycle

1. **Analyze & Research** - Read issue, understand requirements, explore codebase
2. **Issue & Worktree** - Create/update detailed GitHub issue, create linked worktree
3. **Develop & Test** - Implement solution, write tests, verify everything works
4. **Lint** - Fix ALL pre-commit hook issues (see [lint-and-hooks.md](lint-and-hooks.md))
5. **Commit, Push & Merge** - Create PR, squash merge (see [pr-guidelines.md](pr-guidelines.md))
6. **Close & Cleanup** - Close issue, delete worktree

## Why Worktrees?

**Problem with branches:** When multiple agents work in the same repo directory, they constantly switch branches, causing:
- Lost uncommitted work
- File overwrites
- Pre-commit hook failures that reset state
- Context pollution between tasks

**Worktree solution:** Each agent gets an isolated working directory with its own checked-out branch. No conflicts, no lost work.

```
Feature Branch (traditional)     │    Git Worktree
─────────────────────────────────│─────────────────────────────
repo/                            │    repo/           (main)
├── .git/                        │    repo-worktrees/
├── src/  ← only one branch      │    ├── feat-auth/  (feat/auth)
└── ...      at a time           │    ├── feat-api/   (feat/api)
                                 │    └── fix-bug/    (fix/bug-123)
                                 │         ↑ all active simultaneously
```

## Directory Structure

```
~/Documents/GitHub/
├── {repo}/                    # Main repo (main branch, kept clean)
└── {repo}-worktrees/          # All worktrees for this repo
    ├── issue-42/              # Worktree for issue #42
    ├── issue-99/              # Worktree for issue #99
    └── feat-new-api/          # Worktree for feature
```

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

## Worktree Commands

| Action | Command |
|--------|---------|
| List worktrees | `git worktree list` |
| Create worktree | `git worktree add ../{repo}-worktrees/{name} -b {branch}` |
| Remove worktree | `git worktree remove ../{repo}-worktrees/{name}` |
| Prune stale | `git worktree prune` |
| Check current branch | `git branch --show-current` |

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

**Reference:** [lint-and-hooks.md](lint-and-hooks.md) for complete guidance.

```bash
git add . && git commit -m "<type>: <description>"
```

If hooks fail, fix code and retry. Never use `--no-verify`.

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

## Handling Existing Worktrees

```bash
# Check existing worktrees
git worktree list

# If worktree exists, just cd to it
cd ../{repo}-worktrees/issue-{N}

# If branch exists but worktree doesn't
git worktree add ../{repo}-worktrees/issue-{N} feat/{N}-description
```

## Troubleshooting

### "fatal: '{branch}' is already checked out"
Another worktree has this branch. Find it with `git worktree list`.

### Worktree directory already exists
```bash
rm -rf ../{repo}-worktrees/{name}
git worktree prune
git worktree add ../{repo}-worktrees/{name} -b feat/{name}
```

### Lost work after branch switch
This shouldn't happen with worktrees! If you're seeing branch switches, you're in the wrong directory. Always verify with `pwd` and `git branch`.

### Pre-commit blocks commit to main
This is expected! Create a worktree:
```bash
git worktree add ../{repo}-worktrees/my-feature -b feat/my-feature
cd ../{repo}-worktrees/my-feature
# Now you can commit
```

## Best Practices

1. **One worktree per issue** - Don't reuse worktrees for different work
2. **Stay in the worktree** - Don't cd to main repo while working
3. **Commit frequently** - Protect against unexpected issues
4. **Clean up after merge** - Remove worktrees to avoid clutter
5. **Use absolute paths** - When referencing files, use full paths
6. **Keep main clean** - Main repo directory should only have main branch

## Parallel Agent Work

With worktrees, multiple agents can work simultaneously:

```
Agent A: ~/GitHub/repo-worktrees/issue-42/     ← Working on issue 42
Agent B: ~/GitHub/repo-worktrees/issue-99/     ← Working on issue 99
Agent C: ~/GitHub/repo-worktrees/feat-api/     ← Working on API feature
         ↑ All running in parallel, no conflicts
```

Each agent:
- Has its own directory
- Has its own branch checked out
- Cannot interfere with other agents
- Can commit and push independently

## Related Guides

- [Lint & Hooks](lint-and-hooks.md) - Pre-commit hook handling
- [PR Guidelines](pr-guidelines.md) - Pull request creation
- [Git Workflow](git-workflow.md) - Branch and commit conventions
- [GitHub Labels](github-labels.md) - Label creation for issues/PRs

---

*See also: [Git Worktree Documentation](https://git-scm.com/docs/git-worktree)*
