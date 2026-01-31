# Merge Command

**Version:** 1.0.0
**Last Updated:** 2025-01-31

Stage, commit, push, code review, and merge changes in one command. Use `$ARGUMENTS` for an optional commit message.

## Usage

| Command | Action |
|---------|--------|
| `/merge` | Auto-generate commit message, full pipeline |
| `/merge Fix login bug` | Use provided commit message |

## Instructions

When the user runs `/merge`, act as an **orchestrator** coordinating a multi-phase pipeline. Execute each phase in order, stopping on failure unless recovery is possible.

---

### Phase 1: Pre-flight Check

```bash
git status
git diff --stat
```

- If no changes (staged or unstaged) and no unpushed commits, inform the user and stop.
- Detect the current branch. If on `main`:
  - Generate a branch name from the changes (e.g., `feat/update-login-flow`)
  - `git checkout -b <branch-name>`
- Summarize the changes — this context will be passed to the review agent later.

---

### Phase 2: Stage & Commit

#### Stage Files

```bash
git add .
```

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

#### Create Commit

**If `$ARGUMENTS` provided:** Use it as the commit message.

**If no arguments:** Generate a commit message based on the changes:

```
<type>: <short summary>

- Specific change 1
- Specific change 2

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

**Types:** feat, fix, docs, refactor, test, chore, style

```bash
git commit -m "$(cat <<'EOF'
<type>: <summary>

- Change 1
- Change 2

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

**If pre-commit hooks fail:** Fix the issues, re-stage, and retry. NEVER use `--no-verify`.

---

### Phase 3: Push

```bash
git push -u origin <branch>
```

If rejected (remote has new commits):

```bash
git pull --rebase && git push -u origin <branch>
```

---

### Phase 4: Code Review (Sub-Agent)

Spawn a code review agent using the Task tool:

```
Task tool with:
  description: "Code review for merge"
  subagent_type: "general-purpose"
  prompt: |
    You are a code review agent. Review the diff of the current branch against main.

    Target: diff of current branch vs main
    Focus: all
    Severity threshold: medium

    Run: git diff main...HEAD

    Then follow the code review process from .claude/agents/code-review-agent.md
    to produce a structured review report.

    Report findings and stop. Do NOT modify any files.
```

The agent returns a structured review report.

---

### Phase 5: Review Gate

Parse the review report and decide:

**If critical or high severity issues found:**

1. Display the findings to the user
2. Ask using AskUserQuestion:
   - "Fix issues and re-merge?" — Apply fixes, re-commit, re-push, loop back to Phase 4
   - "Merge anyway?" — Proceed to Phase 6
   - "Abort?" — Stop execution
3. If fixing: apply the suggested fixes, `git add . && git commit`, `git push`, then re-run Phase 4

**If no critical or high issues:**

- Proceed automatically to Phase 6
- Display medium/low findings as FYI to the user

---

### Phase 6: Create PR & Merge

Create the pull request with the review summary in the body:

```bash
gh pr create --title "<commit message first line>" --body "$(cat <<'EOF'
## Summary
<bullet points from changes>

## Code Review
<summary of review findings, or "No critical issues found">

---
🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

Then merge:

```bash
gh pr merge --squash --delete-branch
```

If merge fails (e.g., CI checks pending), inform the user and provide the PR URL.

---

### Phase 7: Cleanup

```bash
git checkout main
git pull origin main
```

Report final status:

```
Merged successfully.

Commit: <hash> - <message>
PR: <url>
Review: <X issues found — Y critical, Z high, W medium>
Branch: <branch> (deleted)
```

---

## Common Scenarios

### Nothing to commit or push

```
Working tree clean and up to date. Nothing to merge.
```

### Already on main with no changes

```
No changes detected. Nothing to merge.
```

### Review finds critical issues

Display findings, ask user to fix/merge anyway/abort. If fixing, loop through phases 4-5 again.

### PR merge blocked by CI

```
PR created but merge blocked by CI checks.
PR URL: <url>
Please merge manually after checks pass.
```

---

## Related

- `/push` - Stage, commit, and push (without review or merge)
- `/dev-cycle` - Full autonomous development cycle (for larger changes)
- [code-review-agent](../agents/code-review-agent.md) - Code review agent template
- [git-workflow](../guides/git-workflow.md) - Standard git practices
- [lint-and-hooks](../guides/lint-and-hooks.md) - Pre-commit hook handling
- [pr-guidelines](../guides/pr-guidelines.md) - Pull request creation standards

---

## Version History

### 1.0.0 (2025-01-31)

- Initial implementation
- Full pipeline: stage, commit, push, review, merge
- Orchestrator pattern with code review sub-agent
- Review gate with fix/merge/abort options
- Auto-branch creation when on main
- Reuses existing code-review-agent template
