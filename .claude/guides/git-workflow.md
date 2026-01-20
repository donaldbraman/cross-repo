# Git Workflow Guide

**Version:** 1.0.0
**Last Updated:** 2025-01-20
**Applies to:** All repositories

## Standard Git Workflow

All projects follow this consistent git workflow pattern.

## Branch Strategy

### Visual Overview

```
main (production-ready)
  │
  ├─── feature/442-v3-api-hybrid-search
  │      │
  │      ├─ commit: Add hybrid search service
  │      ├─ commit: Add v3 API endpoint
  │      ├─ commit: Update documentation
  │      │
  │      └─ PR #443 (squash merge) ──> main
  │                                      │
  ├─── feature/445-fix-timeout          │
  │      │                               │
  │      ├─ commit: Add connection pool  │
  │      ├─ commit: Fix N+1 query        │
  │      │                               │
  │      └─ PR #446 (squash merge) ──────┘
  │
  └─── feature/447-add-caching (in progress)
         │
         ├─ commit: Add Redis client
         └─ commit: Implement cache layer
```

### Main Branch
- **Name:** `main` (not master)
- **Protection:** Never commit directly
- **Purpose:** Production-ready code only

### Feature Branches
- **Naming:** `feature/<issue-number>-<short-description>`
- **Example:** `feature/442-v3-api-hybrid-search`
- **Lifespan:** Created from main, merged back, then deleted
- **Commits:** Squashed on merge for clean history

### Branch Creation
```bash
# Always start from main
git checkout main
git pull

# Create feature branch
git checkout -b feature/<issue-number>-<description>
```

## Commit Conventions

### Commit Message Format
```
<type>: <subject>

<body>

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Commit Types
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation only
- `refactor:` Code restructuring (no feature/fix)
- `test:` Adding/updating tests
- `chore:` Tooling, dependencies, maintenance

### Commit Guidelines
- ✅ Present tense ("Add feature" not "Added feature")
- ✅ Imperative mood ("Move cursor to..." not "Moves cursor to...")
- ✅ Focus on "why" not "what" (code shows what)
- ✅ Reference issue numbers
- ❌ Never use `--no-verify`

## Pull Request Workflow

### PR Creation
```bash
# After all commits are made
gh pr create --title "feat: Description" --body "..."

# Merge immediately (autonomous mode)
gh pr merge <number> --squash --delete-branch
```

### PR Requirements
- ✅ Linked to issue
- ✅ Descriptive title matching commit convention
- ✅ Detailed body with:
  - Summary section
  - Test plan
  - Breaking changes (if any)
- ✅ All tests passing
- ✅ Pre-commit hooks passing

### Merge Strategy
- **Method:** Squash merge (creates single commit on main)
- **Timing:** Immediate (autonomous mode)
- **Branch:** Auto-delete after merge

## Common Operations

### Update Feature Branch
```bash
# Rebase on main to stay current
git checkout main
git pull
git checkout feature/xxx
git rebase main
```

### Fix Merge Conflicts
```bash
# Resolve conflicts, then
git add <resolved-files>
git rebase --continue
```

### Abandon Feature Branch
```bash
git checkout main
git branch -D feature/xxx
```

## Pre-commit Hooks

### Hook Behavior
- Runs automatically before each commit
- Performs linting, formatting, type checking
- Cannot be bypassed with `--no-verify` in autonomous mode

### Hook Failure Response
1. **Fix the code** (never bypass)
2. Stage fixes: `git add <files>`
3. Retry commit

### Amend After Hook
If hook modifies files:
```bash
# Check authorship first
git log -1 --format='%an %ae'

# Only amend if you're the author and haven't pushed
git commit --amend --no-edit
```

## Troubleshooting

```bash
# Detached HEAD
git checkout <branch-name>

# Accidentally committed to main
git branch feature/xxx
git reset --hard origin/main
git checkout feature/xxx

# Lost commits
git reflog
git cherry-pick <hash>
```

## Project-Specific Details

This is a **global guide** applicable to all repositories.

Check `docs/guides/git-workflow.md` in current repo for:
- Project-specific branch naming conventions
- Custom merge strategies
- Release branch workflows
- Hotfix procedures

## Related Guides

- [Lint & Hooks](lint-and-hooks.md) - Pre-commit hook handling
- [PR Guidelines](pr-guidelines.md) - Pull request conventions
- [Code Versioning](code-versioning.md) - Version bumping practices
- [Autonomous Cycle](autonomous-cycle.md) - Full development workflow

---

*This guide ensures consistent git practices across all projects.*
