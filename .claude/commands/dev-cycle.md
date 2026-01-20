# Autonomous Development Cycle Command

**Version:** 1.0.1
**Last Updated:** 2025-01-20

Execute a complete autonomous development cycle using worktrees. Use `$ARGUMENTS` to specify the task description or issue number.

## Usage

| Command | Action |
|---------|--------|
| `/dev-cycle Add user authentication` | Create issue and execute full cycle |
| `/dev-cycle #42` | Execute cycle for existing issue #42 |
| `/dev-cycle` | Prompt for task description |

## Instructions

You are executing an autonomous development cycle. Complete all 6 steps without user intervention unless blocked.

---

### Step 1: Analyze & Research

1. **If `$ARGUMENTS` is an issue number (#N):**
   ```bash
   gh issue view N
   ```
   Read the issue and understand requirements.

2. **If `$ARGUMENTS` is a task description:**
   - Parse the description to understand requirements
   - Explore relevant code to understand current implementation
   - Identify affected files and integration points

3. **If `$ARGUMENTS` is empty:**
   - Ask user for task description before proceeding

**Output:** Brief summary of what needs to be done.

---

### Step 2: Issue & Worktree

1. **Create GitHub issue** (skip if already exists):
   ```bash
   gh label create "enhancement" --color "a2eeef" --description "New feature" --force 2>/dev/null || true
   gh label create "bug" --color "d73a4a" --description "Bug fix" --force 2>/dev/null || true

   gh issue create --title "<type>: <description>" --label "<type>" --body "$(cat <<'EOF'
   ## Summary
   [What needs to be done]

   ## Acceptance Criteria
   - [ ] [Criterion 1]
   - [ ] [Criterion 2]

   ## Technical Approach
   [How it will be implemented]
   EOF
   )"
   ```
   Note the issue number N.

2. **Create worktree:**
   ```bash
   cd ~/Documents/GitHub/{repo}
   git checkout main && git pull origin main
   mkdir -p ../{repo}-worktrees
   git worktree add ../{repo}-worktrees/issue-{N} -b feat/{N}-description
   cd ../{repo}-worktrees/issue-{N}
   ```

3. **Verify location:**
   ```bash
   pwd
   git branch --show-current
   ```

**Critical:** All remaining work happens in the worktree directory.

---

### Step 3: Develop & Test

1. **Implement the solution:**
   - Make minimal changes to solve the problem
   - Follow existing code patterns
   - Don't over-engineer

2. **Write or update tests** (if applicable)

3. **Run tests until all pass:**
   ```bash
   # Python
   uv run pytest tests/ -v

   # Node.js
   npm test

   # Or project-specific test command
   ```

4. **Verify functionality works as expected**

**Do not proceed until tests pass.**

---

### Step 4: Lint

**Reference:** [lint-and-hooks](../guides/lint-and-hooks.md) for complete guidance on pre-commit hooks.

1. **Attempt commit:**
   ```bash
   git add .
   git commit -m "<type>: <description>"
   ```

2. **If hooks fail:** Read errors, fix code, retry. Never use `--no-verify`.

**Do not proceed until commit succeeds.**

---

### Step 5: Commit, Push & Merge

1. **Push branch:**
   ```bash
   git push -u origin feat/{N}-description
   ```

2. **Create PR:**
   ```bash
   gh label create "enhancement" --color "a2eeef" --force 2>/dev/null || true

   gh pr create --title "<type>: <description>" --label "<type>" --body "$(cat <<'EOF'
   ## Summary
   - [What changed]
   - [Why it matters]

   ## Testing
   [How it was verified]

   ## Breaking Changes
   None

   Closes #N

   ---
   🤖 Generated with [Claude Code](https://claude.ai/code)
   EOF
   )"
   ```

3. **Merge PR:**
   ```bash
   gh pr merge --squash --delete-branch
   ```

---

### Step 6: Close & Cleanup

1. **Return to main repo:**
   ```bash
   cd ~/Documents/GitHub/{repo}
   ```

2. **Remove worktree:**
   ```bash
   git worktree remove ../{repo}-worktrees/issue-{N}
   ```

3. **Update main:**
   ```bash
   git pull origin main
   git worktree prune
   ```

4. **Verify clean state:**
   ```bash
   git status
   git worktree list
   ```

Issue closes automatically from "Closes #N" in PR body.

---

## Critical Rules

❌ **NEVER** use `--no-verify`
❌ **NEVER** bypass pre-commit hooks
❌ **NEVER** work in main repo directory
❌ **NEVER** skip tests
✅ **ALWAYS** fix hook failures by fixing code
✅ **ALWAYS** stay in worktree until merge
✅ **ALWAYS** clean up worktree after merge

---

## Troubleshooting

### "fatal: '{branch}' is already checked out"
```bash
git worktree list  # Find which worktree has it
```

### Worktree directory already exists
```bash
rm -rf ../{repo}-worktrees/issue-{N}
git worktree prune
git worktree add ../{repo}-worktrees/issue-{N} -b feat/{N}-description
```

### Pre-commit hook fails repeatedly
See [lint-and-hooks](../guides/lint-and-hooks.md) for common fixes and troubleshooting.

### Merge fails
```bash
gh pr checks  # Check what's failing
gh pr view    # View PR status
```

---

## Completion Checklist

Before reporting completion, verify:
- [ ] Issue created (or existing issue used)
- [ ] Worktree created and used for all work
- [ ] Solution implemented
- [ ] Tests pass
- [ ] Lint/hooks pass
- [ ] PR created and merged
- [ ] Issue closed
- [ ] Worktree removed
- [ ] Main branch updated

---

## Related

- [autonomous-cycle](../guides/autonomous-cycle.md) - Detailed guide for worktree-based development
- [lint-and-hooks](../guides/lint-and-hooks.md) - Pre-commit hook handling
- [pr-guidelines](../guides/pr-guidelines.md) - Pull request creation standards
- `/push` - Quick commit and push (for simpler changes)

---

## Version History

### 1.0.1 (2025-01-20)
- Consolidated pre-commit hook guidance to reference lint-and-hooks.md
- Reduced duplication in Step 4 and Troubleshooting sections

### 1.0.0 (2025-01-19)
- Initial release
- Full 6-step autonomous cycle
- Worktree-based workflow
- Troubleshooting section
