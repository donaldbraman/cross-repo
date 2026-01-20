# Troubleshooting Hub

**Version:** 1.0.0
**Last Updated:** 2025-01-20
**Applies to:** All repositories

Central reference for common errors and their solutions across the development workflow.

## Quick Navigation

- [Git & Worktree Errors](#git--worktree-errors)
- [Pre-commit Hook Failures](#pre-commit-hook-failures)
- [GitHub & PR Issues](#github--pr-issues)
- [Agent Messaging Issues](#agent-messaging-issues)

---

## Git & Worktree Errors

See also: [autonomous-cycle.md](guides/autonomous-cycle.md), [git-workflow.md](guides/git-workflow.md)

### "fatal: '{branch}' is already checked out"

**Cause:** Another worktree already has this branch checked out.

**Solution:**
```bash
# Find which worktree has the branch
git worktree list

# Either cd to that worktree or create a new branch
git worktree add ../{repo}-worktrees/{name} -b feat/{new-name}
```

### Worktree directory already exists

**Cause:** Previous worktree was not properly cleaned up.

**Solution:**
```bash
rm -rf ../{repo}-worktrees/{name}
git worktree prune
git worktree add ../{repo}-worktrees/{name} -b feat/{name}
```

### Detached HEAD state

**Cause:** Checked out a commit hash or tag instead of a branch.

**Solution:**
```bash
git checkout <branch-name>
```

### Accidentally committed to main

**Cause:** Made changes directly on main branch instead of feature branch.

**Solution:**
```bash
# Create feature branch with your changes
git branch feature/xxx

# Reset main to match remote
git reset --hard origin/main

# Switch to feature branch
git checkout feature/xxx
```

### Lost commits

**Cause:** Reset, rebase, or branch deletion without preserving commits.

**Solution:**
```bash
# View recent actions (commits are rarely truly lost)
git reflog

# Recover specific commit
git cherry-pick <hash>
```

### Lost work after branch switch

**Cause:** Working in main repo directory instead of worktree.

**Solution:**
This shouldn't happen with worktrees. Always verify location:
```bash
pwd
git branch --show-current
```

If you see branch switches happening, you're in the wrong directory.

### Pre-commit blocks commit to main

**Cause:** Protection hook prevents direct commits to main branch.

**Solution:**
This is expected behavior. Create a worktree for your changes:
```bash
git worktree add ../{repo}-worktrees/my-feature -b feat/my-feature
cd ../{repo}-worktrees/my-feature
# Now you can commit
```

---

## Pre-commit Hook Failures

See also: [lint-and-hooks.md](guides/lint-and-hooks.md)

### Critical Rule

**NEVER** use `--no-verify` to bypass hooks. **ALWAYS** fix the code.

### Hooks not running

**Cause:** Pre-commit not installed or hooks not set up.

**Solution:**
```bash
# Check if hook exists
ls .git/hooks/pre-commit

# Install/reinstall hooks
pre-commit install

# Test hooks manually
pre-commit run --all-files
```

### D103: Missing docstring

**Cause:** Function or class lacks a docstring.

**Solution:**
```python
def search_chunks(query: str, limit: int):
    """Search chunks and return top results."""
    results = db.query(query)
    return results[:limit]
```

### E501: Line too long

**Cause:** Line exceeds maximum length (usually 88 or 120 characters).

**Solution:**
```python
# Break into multiple lines
result = calculate_score(
    query_embedding,
    chunk_embedding,
    boost_factor,
    normalization_constant
)
```

### I001: Import order

**Cause:** Imports not sorted according to style guide.

**Solution:**
```bash
# Auto-fix with ruff
ruff check --fix .
```

### Type errors (mypy)

**Cause:** Missing or incorrect type annotations.

**Solution:**
```python
def get_chunks(ids: list[int]) -> list[Chunk]:
    return db.fetch(ids)
```

### Auto-fix modified files

**Cause:** Hook auto-formatted files but commit was rejected.

**Solution:**
```bash
# If you're the author and haven't pushed yet
git add -A
git commit --amend --no-edit

# Otherwise, create a new commit
git add -A
git commit -m "style: Apply auto-fixes"
```

### Hook fails repeatedly

**Cause:** Underlying code issue not addressed.

**Solution:**
Read the error output carefully. Common patterns:
- Missing docstrings: Add them to all public functions
- Line too long: Break lines or refactor
- Import order: Run `ruff check --fix .`
- Type errors: Add proper type annotations

---

## GitHub & PR Issues

See also: [pr-guidelines.md](guides/pr-guidelines.md)

### PR creation fails

**Cause:** Authentication issue or wrong branch state.

**Solution:**
```bash
# Check authentication
gh auth status

# Verify you're on a feature branch (not main)
git branch --show-current
```

### Can't merge PR

**Cause:** CI checks failing or merge conflicts.

**Solution:**
```bash
# Check CI status
gh pr checks <number>

# View PR details
gh pr view <number>

# If conflicts, update branch
git fetch origin
git rebase origin/main
# Fix conflicts, then force push
git push --force-with-lease
```

### Merge fails

**Cause:** Various issues (permissions, conflicts, failing checks).

**Solution:**
```bash
# Diagnose the problem
gh pr checks  # Check what's failing
gh pr view    # View PR status and details
```

---

## Agent Messaging Issues

See also: [agent-messaging.md](guides/agent-messaging.md)

### Messages not appearing

**Cause:** Labels don't exist or authentication issue.

**Solution:**
```bash
# Check if agent-message label exists
gh label list --repo your-repo | grep agent-message

# Check for messages manually
gh issue list --label "agent-message,to:your-agent" --repo target-repo

# Verify authentication
gh auth status
```

### Can't send messages to another repo

**Cause:** Missing labels in target repo or insufficient permissions.

**Solution:**
```bash
# Check if labels exist in target repo
gh label list --repo target-repo | grep agent-message

# Verify you can access the repo
gh repo view target-repo
```

---

## Related Guides

| Guide | Purpose |
|-------|---------|
| [Autonomous Cycle](guides/autonomous-cycle.md) | Full development workflow with worktrees |
| [Git Workflow](guides/git-workflow.md) | Branch and commit conventions |
| [Lint & Hooks](guides/lint-and-hooks.md) | Pre-commit hook handling |
| [PR Guidelines](guides/pr-guidelines.md) | Pull request creation standards |
| [Agent Messaging](guides/agent-messaging.md) | Agent-to-agent communication |

---

*This hub consolidates troubleshooting from across documentation. For detailed context on any issue, see the related guide.*
