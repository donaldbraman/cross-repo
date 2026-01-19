# Worktree-Based Development Cycle

**Version:** 1.1.0
**Last Updated:** 2025-12-15
**Applies to:** All repositories

## Overview

This guide describes how to use **git worktrees** for development. Worktrees prevent branch-switching conflicts when multiple agents work on the same repository simultaneously.

## Why Worktrees?

**Problem with branches:** When multiple Claude Code agents work in the same repo directory, they constantly switch branches, causing:
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
    ├── feat-auth/             # Worktree for auth feature
    ├── issue-662/             # Worktree for issue #662
    └── fix-bug-123/           # Worktree for bug fix
```

## Development Cycle with Worktrees

### Phase 1: Setup Worktree

```bash
# From main repo directory
cd ~/Documents/GitHub/{repo}

# Ensure main is up to date
git checkout main
git pull origin main

# Create worktrees directory (first time only)
mkdir -p ../{repo}-worktrees

# Create worktree with new branch
git worktree add ../{repo}-worktrees/{feature-name} -b feat/{feature-name}

# Change to worktree directory
cd ../{repo}-worktrees/{feature-name}

# Install dependencies (if using uv)
uv sync

# Verify you're in the right place
pwd
git branch
```

**Examples:**
```bash
# For a feature
git worktree add ../myrepo-worktrees/new-api -b feat/new-api

# For an issue
git worktree add ../myrepo-worktrees/issue-42 -b fix/issue-42

# For a bug fix
git worktree add ../myrepo-worktrees/fix-typo -b fix/typo-in-readme
```

### Phase 2: Develop & Test

Work entirely within the worktree directory:

```bash
# All work happens here
cd ~/Documents/GitHub/{repo}-worktrees/{feature-name}

# Make changes, run tests
uv run pytest tests/ -v

# Commit frequently
git add .
git commit -m "feat: description"
```

**Important:** Never `cd` back to the main repo while working. Stay in the worktree.

### Phase 3: Push & Create PR

```bash
# Push branch to remote
git push -u origin feat/{feature-name}

# Create PR using gh CLI
gh pr create --title "feat: Title" --body "$(cat <<'EOF'
## Summary
- Bullet points

## Test plan
- [ ] Tests pass

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### Phase 4: Merge & Cleanup

```bash
# Merge PR (from worktree directory)
gh pr merge --squash --delete-branch

# Return to main repo
cd ~/Documents/GitHub/{repo}

# Remove worktree
git worktree remove ../{repo}-worktrees/{feature-name}

# Update main
git checkout main
git pull origin main

# Prune any stale worktree references
git worktree prune
```

## Quick Reference Commands

| Action | Command |
|--------|---------|
| List worktrees | `git worktree list` |
| Create worktree | `git worktree add ../{repo}-worktrees/{name} -b {branch}` |
| Remove worktree | `git worktree remove ../{repo}-worktrees/{name}` |
| Prune stale | `git worktree prune` |
| Check current branch | `git branch --show-current` |

## Full Autonomous Cycle Script

For agents, here's the complete flow:

```bash
# === SETUP ===
cd ~/Documents/GitHub/{repo}
git checkout main && git pull origin main
mkdir -p ../{repo}-worktrees
git worktree add ../{repo}-worktrees/{feature} -b feat/{feature}
cd ../{repo}-worktrees/{feature}
uv sync  # if using uv

# === DEVELOP ===
# ... make changes ...
git add . && git commit -m "feat: ..."
uv run pytest tests/ -v

# === SHIP ===
git push -u origin feat/{feature}
gh pr create --title "..." --body "..."
gh pr merge --squash --delete-branch

# === CLEANUP ===
cd ~/Documents/GitHub/{repo}
git worktree remove ../{repo}-worktrees/{feature}
git pull origin main
git worktree prune
```

## Handling Existing Worktrees

```bash
# Check existing worktrees
git worktree list

# If worktree exists, just cd to it
cd ../{repo}-worktrees/{feature}

# If branch exists but worktree doesn't
git worktree add ../{repo}-worktrees/{feature} feat/{feature}
```

## Troubleshooting

### "fatal: '{branch}' is already checked out"
Another worktree has this branch. Find it with `git worktree list`.

### Worktree directory already exists
```bash
rm -rf ../{repo}-worktrees/{feature}
git worktree prune
git worktree add ../{repo}-worktrees/{feature} -b feat/{feature}
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

1. **One worktree per feature/issue** - Don't reuse worktrees for different work
2. **Stay in the worktree** - Don't cd to main repo while working
3. **Commit frequently** - Protect against any unexpected issues
4. **Clean up after merge** - Remove worktrees to avoid clutter
5. **Use absolute paths** - When referencing files, use full paths
6. **Keep main clean** - Main repo directory should only have main branch

## Parallel Agent Work

With worktrees, multiple agents can work simultaneously:

```
Agent A: ~/GitHub/repo-worktrees/feat-auth/    ← Working on auth
Agent B: ~/GitHub/repo-worktrees/feat-search/  ← Working on search
Agent C: ~/GitHub/repo-worktrees/fix-bug-99/   ← Fixing bug
         ↑ All running in parallel, no conflicts
```

Each agent:
- Has its own directory
- Has its own branch checked out
- Cannot interfere with other agents
- Can commit and push independently

## Related Guides

- [Git Workflow](git-workflow.md) - Branch naming and commit conventions
- [Autonomous Cycle](autonomous-cycle.md) - Full development workflow
- [PR Guidelines](pr-guidelines.md) - Pull request standards

---

*See also: [Git Worktree Documentation](https://git-scm.com/docs/git-worktree)*
