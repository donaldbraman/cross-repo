# Worked Example: Autonomous Development Cycle

**Version:** 1.0.0
**Last Updated:** 2025-01-20
**Related:** [/dev-cycle](../commands/dev-cycle.md), [autonomous-cycle guide](../guides/autonomous-cycle.md)

Complete walkthrough of executing a `/dev-cycle` command from start to finish.

---

## Scenario

**Task:** Add a new utility function to validate email addresses in a Python project.

**Repository:** `email-validator` (hypothetical)
**Starting point:** User runs `/dev-cycle Add email validation utility function`

---

## Step 1: Analyze & Research

The agent parses the task description and explores the codebase.

```bash
# Agent examines project structure
$ ls -la
total 32
drwxr-xr-x  8 user  staff   256 Jan 20 10:00 .
drwxr-xr-x  5 user  staff   160 Jan 20 09:00 ..
drwxr-xr-x 12 user  staff   384 Jan 20 10:00 .git
-rw-r--r--  1 user  staff   234 Jan 20 09:00 .pre-commit-config.yaml
-rw-r--r--  1 user  staff   156 Jan 20 09:00 pyproject.toml
drwxr-xr-x  4 user  staff   128 Jan 20 09:00 src
drwxr-xr-x  3 user  staff    96 Jan 20 09:00 tests

# Agent checks existing utilities
$ ls src/utils/
__init__.py
string_utils.py
```

**Agent output:**
> The project has a `src/utils/` directory with existing utility modules. I'll add `email_utils.py` with a validation function following the existing pattern in `string_utils.py`.

---

## Step 2: Issue & Worktree

### Create GitHub Issue

```bash
$ gh label create "enhancement" --color "a2eeef" --description "New feature" --force 2>/dev/null || true

$ gh issue create --title "feat: Add email validation utility function" --label "enhancement" --body "$(cat <<'EOF'
## Summary
Add a utility function to validate email addresses following RFC 5322 standards.

## Acceptance Criteria
- [ ] Create `src/utils/email_utils.py` with `is_valid_email()` function
- [ ] Function validates email format using regex
- [ ] Returns `True` for valid emails, `False` for invalid
- [ ] Includes type hints and docstring
- [ ] Unit tests cover common valid and invalid email patterns

## Technical Approach
1. Create new module `src/utils/email_utils.py`
2. Implement `is_valid_email(email: str) -> bool`
3. Use standard library `re` module for validation
4. Add tests in `tests/test_email_utils.py`
EOF
)"
```

**Output:**
```
https://github.com/user/email-validator/issues/42
```

### Create Worktree

```bash
$ cd ~/Documents/GitHub/email-validator
$ git checkout main && git pull origin main
Already on 'main'
Already up to date.

$ mkdir -p ../email-validator-worktrees
$ git worktree add ../email-validator-worktrees/issue-42 -b feat/42-email-validation
Preparing worktree (new branch 'feat/42-email-validation')
HEAD is now at abc1234 chore: Update dependencies

$ cd ../email-validator-worktrees/issue-42
$ pwd
/Users/user/Documents/GitHub/email-validator-worktrees/issue-42

$ git branch --show-current
feat/42-email-validation
```

---

## Step 3: Develop & Test

### Implement the Solution

**Create `src/utils/email_utils.py`:**

```python
"""Email validation utilities."""

import re


def is_valid_email(email: str) -> bool:
    """Validate an email address format.

    Args:
        email: The email address to validate.

    Returns:
        True if the email format is valid, False otherwise.

    Examples:
        >>> is_valid_email("user@example.com")
        True
        >>> is_valid_email("invalid-email")
        False
    """
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))
```

**Create `tests/test_email_utils.py`:**

```python
"""Tests for email validation utilities."""

import pytest

from src.utils.email_utils import is_valid_email


class TestIsValidEmail:
    """Tests for is_valid_email function."""

    @pytest.mark.parametrize(
        "email",
        [
            "user@example.com",
            "user.name@example.com",
            "user+tag@example.com",
            "user@subdomain.example.com",
        ],
    )
    def test_valid_emails(self, email: str) -> None:
        """Test that valid emails return True."""
        assert is_valid_email(email) is True

    @pytest.mark.parametrize(
        "email",
        [
            "invalid",
            "invalid@",
            "@example.com",
            "user@.com",
            "user@example",
            "",
        ],
    )
    def test_invalid_emails(self, email: str) -> None:
        """Test that invalid emails return False."""
        assert is_valid_email(email) is False
```

### Run Tests

```bash
$ uv run pytest tests/test_email_utils.py -v
========================= test session starts ==========================
platform darwin -- Python 3.12.0, pytest-8.0.0
collected 10 items

tests/test_email_utils.py::TestIsValidEmail::test_valid_emails[user@example.com] PASSED
tests/test_email_utils.py::TestIsValidEmail::test_valid_emails[user.name@example.com] PASSED
tests/test_email_utils.py::TestIsValidEmail::test_valid_emails[user+tag@example.com] PASSED
tests/test_email_utils.py::TestIsValidEmail::test_valid_emails[user@subdomain.example.com] PASSED
tests/test_email_utils.py::TestIsValidEmail::test_invalid_emails[invalid] PASSED
tests/test_email_utils.py::TestIsValidEmail::test_invalid_emails[invalid@] PASSED
tests/test_email_utils.py::TestIsValidEmail::test_invalid_emails[@example.com] PASSED
tests/test_email_utils.py::TestIsValidEmail::test_invalid_emails[user@.com] PASSED
tests/test_email_utils.py::TestIsValidEmail::test_invalid_emails[user@example] PASSED
tests/test_email_utils.py::TestIsValidEmail::test_invalid_emails[] PASSED

========================== 10 passed in 0.12s ==========================
```

---

## Step 4: Lint

### First Commit Attempt

```bash
$ git add .
$ git commit -m "feat: Add email validation utility function"
```

**Output (hook failure):**

```
ruff.....................................................................Failed
- hook id: ruff
- exit code: 1

src/utils/email_utils.py:1:1: D100 Missing docstring in public module
Found 1 error.

mypy.....................................................................Passed
pytest...................................................................Passed
```

### Fix the Issue

The file has a docstring, but it's after the import. Move the docstring to the top:

```python
"""Email validation utilities."""

import re
# ... rest of file
```

Wait, the file already has this. Let me check the actual error more carefully. The hook says D100, which means the module docstring might not be recognized. Let me verify the file doesn't have any whitespace before the docstring:

```bash
$ head -1 src/utils/email_utils.py | cat -A
"""Email validation utilities."""$
```

The docstring is correct. This might be a false positive. Check if the hook auto-fixed:

```bash
$ git status
On branch feat/42-email-validation
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   src/utils/email_utils.py
        new file:   tests/test_email_utils.py
```

### Retry Commit

```bash
$ git commit -m "feat: Add email validation utility function"
ruff.....................................................................Passed
mypy.....................................................................Passed
pytest...................................................................Passed
[feat/42-email-validation abc5678] feat: Add email validation utility function
 2 files changed, 52 insertions(+)
 create mode 100644 src/utils/email_utils.py
 create mode 100644 tests/test_email_utils.py
```

---

## Step 5: Commit, Push & Merge

### Push Branch

```bash
$ git push -u origin feat/42-email-validation
Enumerating objects: 8, done.
Counting objects: 100% (8/8), done.
Delta compression using up to 10 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (7/7), 1.23 KiB | 1.23 MiB/s, done.
Total 7 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 1 local object.
remote:
remote: Create a pull request for 'feat/42-email-validation' on GitHub by visiting:
remote:      https://github.com/user/email-validator/pull/new/feat/42-email-validation
remote:
To https://github.com/user/email-validator.git
 * [new branch]      feat/42-email-validation -> feat/42-email-validation
Branch 'feat/42-email-validation' set up to track remote branch 'feat/42-email-validation' from 'origin'.
```

### Create PR

```bash
$ gh pr create --title "feat: Add email validation utility function" --label "enhancement" --body "$(cat <<'EOF'
## Summary
- Added `is_valid_email()` function in `src/utils/email_utils.py`
- Validates email format using RFC 5322-compatible regex pattern
- Includes comprehensive type hints and docstrings

## Testing
```bash
uv run pytest tests/test_email_utils.py -v
# 10 tests passed
```

Test coverage includes:
- Valid email formats (standard, with dots, with tags, subdomains)
- Invalid email formats (missing @, missing domain, empty string)

## Breaking Changes
None

Closes #42

---
Generated with [Claude Code](https://claude.ai/code)
EOF
)"
```

**Output:**
```
https://github.com/user/email-validator/pull/43
```

### Merge PR

```bash
$ gh pr merge --squash --delete-branch
Merging pull request #43 (feat: Add email validation utility function)
Deleted branch 'feat/42-email-validation' on GitHub.
```

---

## Step 6: Close & Cleanup

### Return to Main Repo

```bash
$ cd ~/Documents/GitHub/email-validator
```

### Remove Worktree

```bash
$ git worktree remove ../email-validator-worktrees/issue-42
```

### Update Main

```bash
$ git pull origin main
remote: Enumerating objects: 1, done.
remote: Counting objects: 100% (1/1), done.
remote: Total 1 (delta 0), reused 0 (delta 0)
Unpacking objects: 100% (1/1), done.
From https://github.com/user/email-validator
 * branch            main       -> FETCH_HEAD
   abc1234..def5678  main       -> origin/main
Updating abc1234..def5678
Fast-forward
 src/utils/email_utils.py   | 26 ++++++++++++++++++++++++++
 tests/test_email_utils.py  | 26 ++++++++++++++++++++++++++
 2 files changed, 52 insertions(+)
 create mode 100644 src/utils/email_utils.py
 create mode 100644 tests/test_email_utils.py

$ git worktree prune
```

### Verify Clean State

```bash
$ git status
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean

$ git worktree list
/Users/user/Documents/GitHub/email-validator  abc5678 [main]
```

---

## What Success Looks Like

After completing the cycle:

1. **Issue #42 is closed** - Automatically closed by "Closes #42" in PR body
2. **PR #43 is merged** - Squash-merged to main
3. **Code is on main** - `src/utils/email_utils.py` and `tests/test_email_utils.py` exist
4. **Tests pass** - All 10 tests pass in CI
5. **Worktree is removed** - Only main repo directory exists
6. **Main is up to date** - Latest changes pulled

---

## Common Variations

### Working with an Existing Issue

```bash
$ /dev-cycle #42
# Agent reads issue #42 and skips issue creation
$ gh issue view 42
# ... reads requirements and proceeds to worktree creation
```

### Handling Test Failures

If tests fail in Step 3:

```bash
$ uv run pytest tests/test_email_utils.py -v
FAILED tests/test_email_utils.py::TestIsValidEmail::test_valid_emails[user+tag@example.com]

# Agent identifies the issue (regex doesn't handle + correctly)
# Agent fixes the regex pattern
# Agent re-runs tests until all pass
```

### Pre-commit Hook Auto-Fixes

If hooks auto-fix files (e.g., ruff formats code):

```bash
$ git commit -m "feat: Add feature"
ruff.....................................................................Failed
- hook id: ruff-format
- files were modified by this hook

# Files were auto-formatted. Stage the changes:
$ git add -A
$ git commit -m "feat: Add feature"
# Second attempt succeeds because files are now formatted
```

### Branch Already Exists

```bash
$ git worktree add ../email-validator-worktrees/issue-42 -b feat/42-email-validation
fatal: 'feat/42-email-validation' is already checked out at '...'

# Solution: Check if worktree exists
$ git worktree list
/Users/user/.../issue-42  abc1234 [feat/42-email-validation]

# Either cd to existing worktree or remove it first
$ cd ../email-validator-worktrees/issue-42
# OR
$ git worktree remove ../email-validator-worktrees/issue-42
$ git branch -D feat/42-email-validation
$ git worktree add ../email-validator-worktrees/issue-42 -b feat/42-email-validation
```

---

## Edge Cases

### No Tests Needed

For documentation-only changes:

```bash
# Step 3 becomes: verify documentation renders correctly
$ grip README.md --export README.html
$ open README.html
# Visual inspection passes
```

### Merge Conflicts

If the PR has merge conflicts (rare with worktrees):

```bash
$ gh pr merge --squash --delete-branch
! Pull request #43 is not mergeable: the base branch is out of date

# Solution: Rebase on main
$ git fetch origin main
$ git rebase origin/main
# Resolve any conflicts
$ git push --force-with-lease
$ gh pr merge --squash --delete-branch
```

### CI Failures

If CI checks fail after PR creation:

```bash
$ gh pr checks 43
Some checks were not successful
1 failing, 0 pending, 3 passing

$ gh pr checks 43 --watch
# Wait for details

# Fix the issue in the worktree
$ git add .
$ git commit -m "fix: Address CI feedback"
$ git push
# CI re-runs
```

---

## Related Examples

- [correction-workflow-example.md](correction-workflow-example.md) - Running /proof and applying corrections
