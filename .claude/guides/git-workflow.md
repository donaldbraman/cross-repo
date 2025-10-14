# Git Workflow Guide

**Version:** 1.0.0
**Last Updated:** 2025-10-14

## Standard Git Workflow

All projects follow this consistent git workflow pattern.

## Branch Strategy

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

🤖 Generated with [Claude Code](https://claude.com/claude-code)

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

## Best Practices

### Before Starting Work
1. ✅ Pull latest main
2. ✅ Create issue
3. ✅ Create feature branch from main
4. ✅ Make small, focused commits

### During Development
1. ✅ Commit frequently
2. ✅ Write descriptive commit messages
3. ✅ Let hooks run (never bypass)
4. ✅ Rebase on main regularly

### After Completing Work
1. ✅ Run all tests
2. ✅ Verify pre-commit hooks pass
3. ✅ Create PR
4. ✅ Merge immediately (autonomous mode)
5. ✅ Delete feature branch
6. ✅ Close issue

## Troubleshooting

### Detached HEAD State
```bash
# Return to branch
git checkout <branch-name>
```

### Accidentally Committed to Main
```bash
# Move commits to feature branch
git branch feature/xxx
git reset --hard origin/main
git checkout feature/xxx
```

### Lost Commits
```bash
# Find commit hash
git reflog

# Recover commit
git cherry-pick <hash>
```

---

*This guide ensures consistent git practices across all projects.*
