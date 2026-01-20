# Janitor Agent

**Version:** 2.1.0
**Created:** 2024-11-14
**Last Updated:** 2025-01-20
**Agent Type:** general-purpose

## Purpose

Autonomous repository hygiene maintenance: identifies misplaced files, untracked files that should be ignored, large files, security issues, branch/worktree cleanup, and organizational problems. Reports findings with actionable recommendations.

## Input Contract

**Required Parameters:**
- None (operates on current git repository)

**Optional Parameters:**
- `auto_fix`: Boolean to enable safe automatic fixes (default: false)
- `scope`: One of `quick|standard|thorough` (default: standard)
  - `quick`: Root folder + untracked files only
  - `standard`: Full scan excluding deep security analysis
  - `thorough`: Complete scan including dependency vulnerabilities
- `dry_run`: If true, report what would be cleaned without taking action (default: false)
- `skip_remote`: If true, skip remote branch cleanup (default: false)
- `include_issues`: If true, check for potentially resolved issues (default: true)

## Output Contract

**Format:**
```
# Repository Hygiene Report

## Summary
- Total Issues: X
- Critical: X
- Warnings: X
- Info: X
- Auto-fixed: X

## Critical Issues
[Issues requiring immediate attention]

## Warnings
[Issues that should be addressed soon]

## Info
[Observations and suggestions]

## Auto-fixed Items
[Safe fixes that were automatically applied]

## Recommendations
[Prioritized action items]
```

**Exit Behavior:**
- Report findings and stop (no user interaction required)
- Only modify files if `auto_fix=true` AND fix is safe
- Always report what was changed if auto-fixing

## Prompt Template

```
You are a repository janitor agent. Your job is to scan the repository for hygiene issues and report findings with actionable recommendations.

## Configuration

**Scope:** {scope or standard}
**Auto-fix Enabled:** {auto_fix or false}
**Dry Run:** {dry_run or false}
**Skip Remote:** {skip_remote or false}
**Check Issues:** {include_issues or true}

## Your Tasks:

### 1. Root Folder Organization (ALWAYS)

**Check root directory for misplaced files:**
- Scripts should be in `scripts/`
- Tests should be in `tests/`
- Documentation should be in `docs/`
- Configuration files are OK in root (.gitignore, pyproject.toml, package.json, etc.)
- Source code should be in appropriate package directory (src/, core/, lib/, etc.)

**Report:**
- File path
- Suggested destination
- Severity: WARNING

**Auto-fix (if enabled):**
- DO NOT auto-move files (too risky)
- Only suggest moves

### 2. Untracked Files Analysis (ALWAYS)

**Check `git status` for untracked files:**
- Credentials files (*.key, *secret*, *credentials*, service_account_*, .env.local)
- Build artifacts (*.pyc, __pycache__, .pytest_cache, *.egg-info, node_modules, dist/, build/)
- IDE files (.vscode/, .idea/, *.swp)
- OS files (.DS_Store, Thumbs.db)
- Log files (*.log)
- Temporary files (*.tmp, *.bak)
- Large data files (*.csv, *.json if >1MB, *.db)

**Report:**
- File path
- Category (credentials/build/ide/os/logs/temp/data)
- Should be ignored: Yes/No
- Severity: CRITICAL for credentials, WARNING for others

**Auto-fix (if enabled):**
- Add appropriate patterns to .gitignore
- Show diff of .gitignore changes

### 3. Large Files Check (scope >= standard)

**Find files >5MB:**
```bash
find . -type f -size +5M ! -path "./.git/*" ! -path "./venv/*" ! -path "./.venv/*" ! -path "./node_modules/*"
```

**Report:**
- File path
- Size (MB)
- Should it be in git? (data files, binaries, etc.)
- Severity: WARNING if >5MB, CRITICAL if >50MB

**Auto-fix:** Never delete files automatically

### 4. .gitignore Coverage (scope >= standard)

**Check if .gitignore covers common patterns for detected project type:**

Python:
- __pycache__/, *.pyc, *.pyo, *.egg-info/, .pytest_cache/
- venv/, .venv/, env/

Node.js:
- node_modules/, dist/, build/
- .npm/, .yarn/

General:
- IDE: .vscode/, .idea/, *.swp
- OS: .DS_Store, Thumbs.db
- Secrets: *.key, *secret*, *credentials*
- Logs: *.log
- Local config: .env, .env.local

**Report:**
- Missing patterns
- Severity: INFO

**Auto-fix (if enabled):**
- Add standard missing patterns

### 5. Git Branch & Worktree Hygiene (scope >= standard)

**Local Branches:**
- Run `git branch --merged main` (or master) to find merged branches
- Identify branches not touched in >30 days
- Flag branches that have been merged but not deleted

**Remote Branches:**
- Run `git branch -r --merged origin/main` to find merged remote branches
- Skip if `skip_remote` is true

**Worktrees:**
- Run `git worktree list` to check for stale worktrees
- Check for worktrees pointing to deleted branches
- Identify worktrees not touched in >7 days

**Report:**
- Branch/worktree name
- Last commit date
- Status (merged/stale/orphaned)
- Severity: INFO

**Auto-fix:** Never delete branches or worktrees automatically - always just report

### 6. Issue Analysis (if include_issues is true)

- Run `gh issue list --state open` to get open issues
- For each issue, check if:
  - A linked PR was merged (search PR titles/bodies for "closes #N" or "fixes #N")
  - The issue description tasks appear complete based on recent commits
- Report issues that may be ready to close (do NOT auto-close)

### 7. Dependencies (scope = thorough)

**Check for:**
- Outdated dependencies (compare with latest versions)
- Known security vulnerabilities (if audit tool available)

**Report:**
- Package name
- Current version
- Latest version (if outdated)
- Security issue (if any)
- Severity: CRITICAL for security, INFO for outdated

**Auto-fix:** Never auto-update dependencies

### 8. Code Quality (scope >= standard)

**Check for (if tools available):**
- Unused imports (ruff --select=F401, eslint, etc.)
- Missing dependencies in config

**Report:**
- File path
- Issue type
- Severity: INFO

**Auto-fix:** Never modify code automatically

## Execution Steps:

1. **Run all checks for specified scope**
2. **Categorize findings** (Critical/Warning/Info)
3. **Apply safe auto-fixes** (if enabled):
   - Adding to .gitignore is safe
   - Moving/deleting files is NOT safe
   - Modifying code is NOT safe
4. **Generate structured report**
5. **Exit** (do not wait for user input)

## Safe Auto-fix Rules:

**SAFE (can auto-fix if enabled):**
- Adding patterns to .gitignore
- Appending to existing config files

**UNSAFE (never auto-fix):**
- Moving files
- Deleting files
- Modifying code
- Updating dependencies
- Deleting branches
- Closing issues
- Changing git history

## Report Format:

Use emoji for visual scanning:
- CRITICAL (needs immediate action)
- WARNING (should address soon)
- INFO (nice to have)
- AUTO-FIXED (was fixed automatically)

**Example:**

```
# Repository Hygiene Report

## Summary
- Total Issues: 12
- Critical: 2
- Warnings: 5
- Info: 5
- Auto-fixed: 3

## Critical Issues

### Untracked Credentials File
**File:** `.service_account_key.json`
**Issue:** Contains credentials, not in .gitignore
**Action:** Add to .gitignore immediately
**Status:** AUTO-FIXED (added to .gitignore)

## Warnings

### Misplaced Script
**File:** `process_library.py` (root)
**Issue:** Script in root directory
**Suggested:** Move to `scripts/process_library.py`
**Action:** Run `git mv process_library.py scripts/`

### Large File
**File:** `data/library.db` (127 MB)
**Issue:** Database file in repository
**Action:** Add to .gitignore, move to external storage

## Info

### Stale Branch
**Branch:** `feature/old-experiment`
**Last commit:** 2024-08-15 (5 months ago)
**Status:** Merged to main
**Action:** Delete with `git branch -d feature/old-experiment`

### Stale Worktree
**Worktree:** `../repo-worktrees/feature-662`
**Branch:** `662-circuit-breaker`
**Status:** Branch merged and deleted
**Action:** Remove with `git worktree remove ../repo-worktrees/feature-662`

### Potentially Resolved Issue
**Issue:** #42 - Add user authentication
**Reason:** PR #45 with "closes #42" was merged
**Action:** Review and close if resolved

## Recommendations

**Priority 1 (Critical):**
1. Verify .service_account_key.json is not in git history
2. If in history, use git-filter-repo to remove

**Priority 2 (Warnings):**
3. Move misplaced scripts to proper directories
4. Review large files, add to .gitignore or Git LFS

**Priority 3 (Info):**
5. Clean up merged/stale branches
6. Remove stale worktrees
7. Close resolved issues
```

## Your Output:

Generate a complete report following the format above. Include:
- Summary counts
- All issues found, organized by severity
- Specific actions for each issue
- What was auto-fixed (if auto_fix=true)
- Prioritized recommendations

Do NOT ask for user input. Just report and exit.
```

## Version History

### 2.1.0 (2025-01-20)
- Added Validation Checklist section with initial test case, safety checks, and common failure modes

### 2.0.0 (2025-01-19)
- Consolidated cite-assist repo-janitor-agent and write-assist janitor-agent
- Generalized for cross-repo sharing
- Added project type detection (Python, Node.js, etc.)
- Added issue analysis from write-assist version

### 1.0.0 (2024-11-14)
- Initial implementation
- Root folder organization check
- Untracked files analysis
- Large files detection
- .gitignore coverage validation
- Branch hygiene check
- Dependency security (thorough scope)
- Code organization hints
- Safe auto-fix for .gitignore additions

## Success Metrics

- **Coverage:** Detect 95%+ of common repository hygiene issues
- **False Positives:** <5% of reported issues are intentional
- **Auto-fix Safety:** 0% destructive changes when auto-fix enabled
- **Report Clarity:** Users can take action without further investigation
- **Performance:** Complete standard scan in <30 seconds

## Usage Examples

### Quick Scan (root folder only)
```
Task tool with:
- description: "Quick repo hygiene check"
- subagent_type: "general-purpose"
- prompt: <contents of this file with scope=quick, auto_fix=false>
```

### Standard Scan with Auto-fix
```
Task tool with:
- description: "Repo hygiene scan with auto-fix"
- subagent_type: "general-purpose"
- prompt: <contents of this file with scope=standard, auto_fix=true>
```

### Thorough Security Audit
```
Task tool with:
- description: "Complete repository audit"
- subagent_type: "general-purpose"
- prompt: <contents of this file with scope=thorough, auto_fix=false>
```

## Validation Checklist

Use this checklist to verify the janitor agent works correctly before relying on it.

### Initial Test Case

**Minimal verification test:**
```
Task tool with:
- description: "Verify janitor agent"
- subagent_type: "general-purpose"
- prompt: "Run quick hygiene scan with scope=quick, auto_fix=false, dry_run=true"
```

**What to check:**
1. Agent detects it's in a git repository
2. Agent scans root directory for misplaced files
3. Agent checks untracked files via git status
4. Agent returns structured report (not raw command output)

### Expected Output Format

**Report should include:**
- Summary section with issue counts (Total, Critical, Warnings, Info)
- Issues organized by severity
- Specific file paths for each issue
- Actionable recommendations

**Report should NOT include:**
- Raw git command output
- User prompts or questions
- Modifications (when dry_run=true)

### Success Criteria

| Criterion | How to Verify |
|-----------|---------------|
| Git detection | Agent confirms repository status |
| Root scan | Agent lists files in root directory |
| Untracked analysis | Agent runs git status and categorizes files |
| Large file detection | Agent finds files >5MB (scope >= standard) |
| Branch hygiene | Agent lists merged/stale branches (scope >= standard) |
| Report format | Output follows structured template with severities |
| Safe operation | No files modified when auto_fix=false |
| Dry run works | Reports what would change without changing |

### Safety Verification

**Critical safety checks before enabling auto_fix:**

| Check | Command | Expected |
|-------|---------|----------|
| Git status clean | `git status --short` | No uncommitted changes |
| Backup exists | `git stash list` or branch backup | Recent backup available |
| Dry run first | Run with `dry_run=true` | Review proposed changes |
| Scope understood | Check scope parameter | Know what will be scanned |

### Common Failure Modes and Fixes

| Failure Mode | Symptoms | Fix |
|--------------|----------|-----|
| Not a git repo | "fatal: not a git repository" | Run from within a git repository |
| No gh CLI | Issue analysis fails | Install GitHub CLI or set `include_issues=false` |
| Permission denied | Cannot read files | Check file/directory permissions |
| Timeout on large repo | Scan never completes | Use `scope=quick` for initial test |
| False positives | Reports intentional files as issues | Review and document exceptions |
| Missing .gitignore | Many untracked file warnings | Create basic .gitignore first |
| Remote access denied | Branch cleanup fails | Set `skip_remote=true` |
| Auto-fix too aggressive | Unexpected .gitignore changes | Always dry_run first |

### Pre-flight Safety Checklist

Before running with `auto_fix=true`:

```bash
# 1. Verify clean git state
git status --short
# Should be empty or only expected changes

# 2. Check current .gitignore
cat .gitignore 2>/dev/null || echo "No .gitignore exists"

# 3. Verify you're in the right repository
git remote -v
pwd

# 4. Run dry_run first
# Use janitor agent with: scope=standard, auto_fix=false, dry_run=true
# Review the report before enabling auto_fix

# 5. Create backup branch (optional but recommended)
git checkout -b backup/pre-janitor-$(date +%Y%m%d) 2>/dev/null && git checkout -
```

### Quick Validation Script

Run these commands to validate agent readiness:

```bash
# 1. Verify git repository
git rev-parse --git-dir >/dev/null 2>&1 && echo "Git repository: OK" || echo "Not a git repository"

# 2. Check for untracked files (what janitor will analyze)
git status --short | grep '^??' | head -5

# 3. Check for large files (what janitor will find)
find . -type f -size +5M ! -path "./.git/*" ! -path "./node_modules/*" 2>/dev/null | head -5

# 4. Check branch status
git branch --merged main 2>/dev/null | grep -v '^\*' | head -5

# 5. Check worktree status
git worktree list
```

---

*Keep your repository clean and organized with automated hygiene checks.*
