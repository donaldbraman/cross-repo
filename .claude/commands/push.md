# Push Command

**Version:** 1.0.0
**Last Updated:** 2025-01-19

Stage all changes, create a commit with a descriptive message, and push to remote. Use `$ARGUMENTS` for an optional commit message.

## Usage

| Command | Action |
|---------|--------|
| `/push` | Auto-generate commit message from changes |
| `/push Fix typo in README` | Use provided commit message |

## Instructions

When the user runs `/push`, perform the following steps:

### Step 1: Check Status

```bash
git status
git diff --stat
```

If there are no changes (staged or unstaged), inform the user and stop.

### Step 2: Review Changes

Briefly summarize the key changes:
- Modified files
- New untracked files
- Deleted files

### Step 3: Stage Files

```bash
git add .
```

This stages all changes and respects `.gitignore`.

**Check for files that should be gitignored but aren't:**
- `.env`, credentials, API keys
- Build artifacts (`node_modules/`, `__pycache__/`, `site_libs/`)
- IDE files (`.idea/`, `.vscode/`)
- OS files (`.DS_Store`, `Thumbs.db`)
- Temp files (`*.tmp`, `temp/`)

If found, add them to `.gitignore` first:
```bash
echo "pattern" >> .gitignore
git add .gitignore
```

### Step 4: Create Commit

**If `$ARGUMENTS` provided:** Use it as the commit message.

**If no arguments:** Generate a commit message based on the changes:

```
<type>: <short summary>

- Specific change 1
- Specific change 2

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

**Types:** feat, fix, docs, refactor, test, chore, style

**Guidelines:**
- First line: 50 chars or less, imperative mood
- Body: bullet points for specific changes
- Always end with Co-Authored-By

```bash
git commit -m "$(cat <<'EOF'
<type>: <summary>

- Change 1
- Change 2

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

### Step 5: Push

```bash
git push
```

If rejected (remote has new commits):
```bash
git pull --rebase && git push
```

### Step 6: Verify

```bash
git status
```

Confirm working tree is clean.

### Step 7: Report

```
Committed and pushed.

Commit: <hash> - <message>

Changes:
- <file1>: <what changed>
- <file2>: <what changed>
```

---

## Common Scenarios

### Nothing to commit
```
Working tree clean. Nothing to push.
```

### Pre-commit hooks fail
Fix the issues (see [lint-and-hooks](../guides/lint-and-hooks.md)), then retry:
```bash
git add .
git commit -m "..."
```

### Push rejected
```bash
git pull --rebase
# Resolve conflicts if any
git push
```

### Want to exclude a file
```bash
git reset HEAD <file>  # Unstage specific file
git add .
git commit -m "..."
```

---

## Version History

### 1.0.0 (2025-01-19)
- Initial cross-repo release
- Generalized from criminal-law and jil-seminar versions
- Added $ARGUMENTS support for custom commit messages
- Added .gitignore check for sensitive files
