# Glossary

**Version:** 1.0.0
**Last Updated:** 2025-01-20
**Applies to:** All repositories

This glossary defines key terms used throughout the documentation. Use these definitions to ensure consistent understanding across all guides, commands, and agent configurations.

---

## Terms

### Autonomous Mode

**Definition:** An operational state where an AI agent executes tasks without user intervention, making decisions and completing workflows independently.

**Context:** Autonomous mode is used during the [dev-cycle](commands/dev-cycle.md) when the agent performs all six steps (analyze, issue creation, development, lint, merge, cleanup) without pausing for user input.

**Key constraints:**
- Must NEVER use `--no-verify` to bypass quality gates
- Must NEVER skip pre-commit hooks
- Must complete entire workflow or stop if blocked

**See also:** [Autonomous Cycle Guide](guides/autonomous-cycle.md)

---

### Dev-Cycle

**Definition:** The complete autonomous development workflow consisting of six sequential steps that take a task from conception to merged code.

**The 6 steps:**
1. **Analyze & Research** - Understand requirements, explore codebase
2. **Issue & Worktree** - Create GitHub issue, set up isolated worktree
3. **Develop & Test** - Implement solution, write/run tests
4. **Lint** - Fix all pre-commit hook issues
5. **Commit, Push & Merge** - Create PR, squash merge
6. **Close & Cleanup** - Remove worktree, update main

**Usage:**
```bash
/dev-cycle Add user authentication
/dev-cycle #42
```

**See also:** [Dev-Cycle Command](commands/dev-cycle.md), [Autonomous Cycle Guide](guides/autonomous-cycle.md)

---

### Pre-commit Hooks

**Definition:** Automated scripts that run before each git commit to enforce code quality standards, formatting rules, and security checks.

**Common hooks:**
- **Linters:** ruff (Python), eslint (JavaScript/TypeScript)
- **Formatters:** black, prettier
- **Type checkers:** mypy, tsc
- **Security:** detect-secrets
- **Other:** trailing-whitespace, end-of-file-fixer

**Critical rule:** Never bypass hooks with `--no-verify`. Always fix the underlying code issues.

**Example failure and fix:**
```bash
# Hook reports: D103 Missing docstring in public function
# Fix: Add docstring to the function
def search_chunks(query: str) -> list[Chunk]:
    """Search database and return matching chunks."""
    return db.search(query)
```

**See also:** [Lint & Hooks Guide](guides/lint-and-hooks.md)

---

### Semantic Versioning

**Definition:** A versioning scheme using three numbers (MAJOR.MINOR.PATCH) that conveys meaning about the underlying changes.

**Format:** `MAJOR.MINOR.PATCH` (e.g., `2.1.3`)

**When to increment:**
- **MAJOR** - Breaking changes (incompatible API changes)
- **MINOR** - New features (backward-compatible additions)
- **PATCH** - Bug fixes (backward-compatible fixes)

**Examples:**
```
1.0.0 -> 2.0.0  # Breaking change (removed endpoint)
1.0.0 -> 1.1.0  # New feature (added optional parameter)
1.0.0 -> 1.0.1  # Bug fix (corrected calculation error)
```

**See also:** [Code Versioning Guide](guides/code-versioning.md), [Semantic Versioning Specification](https://semver.org/)

---

### Squash Merge

**Definition:** A git merge strategy that combines all commits from a feature branch into a single commit when merging to the target branch.

**Benefits:**
- Clean, linear commit history on main branch
- Each PR becomes one atomic commit
- Easier to revert entire features if needed
- Cleaner `git log` output

**Usage:**
```bash
gh pr merge --squash --delete-branch
```

**Visual representation:**
```
Feature branch:          main after squash merge:
  commit 1: Add tests       Single commit:
  commit 2: Fix typo   ->   "feat: Add authentication (#42)"
  commit 3: Add docs
```

**See also:** [Git Workflow Guide](guides/git-workflow.md), [PR Guidelines](guides/pr-guidelines.md)

---

### Subagent

**Definition:** An AI agent spawned by a parent agent to handle a specific, delegated task. The subagent operates independently but reports results back to the parent.

**Use cases:**
- Running tests while parent continues other work
- Performing code analysis in parallel
- Handling specialized tasks (e.g., documentation review)

**Characteristics:**
- Has its own context and task scope
- Can operate in its own worktree
- Returns results to parent agent
- May have different capabilities or permissions

**See also:** [Agent Messaging Protocol](guides/agent-messaging.md)

---

### Worktree

**Definition:** A git feature that allows multiple working directories to be linked to a single repository, each with its own checked-out branch.

**Why use worktrees:**
- **Isolation:** Each task gets its own directory, preventing conflicts
- **Parallelism:** Multiple agents can work simultaneously
- **Safety:** No accidental branch switches or lost work

**Directory structure:**
```
~/Documents/GitHub/
├── my-repo/                    # Main repo (main branch)
└── my-repo-worktrees/          # Worktrees directory
    ├── issue-42/               # Working on issue #42
    ├── issue-99/               # Working on issue #99
    └── feat-new-api/           # Feature development
```

**Common commands:**
```bash
# Create worktree with new branch
git worktree add ../repo-worktrees/issue-42 -b feat/42-description

# List all worktrees
git worktree list

# Remove worktree after merge
git worktree remove ../repo-worktrees/issue-42

# Clean up stale references
git worktree prune
```

**See also:** [Autonomous Cycle Guide](guides/autonomous-cycle.md), [Git Worktree Documentation](https://git-scm.com/docs/git-worktree)

---

## Related Guides

- [Autonomous Cycle](guides/autonomous-cycle.md) - Full development workflow
- [Code Versioning](guides/code-versioning.md) - Version management
- [Git Workflow](guides/git-workflow.md) - Branch and commit conventions
- [Lint & Hooks](guides/lint-and-hooks.md) - Pre-commit hook handling
- [PR Guidelines](guides/pr-guidelines.md) - Pull request standards
- [Agent Messaging](guides/agent-messaging.md) - Agent communication protocol

---

*This glossary ensures consistent terminology across all documentation.*
