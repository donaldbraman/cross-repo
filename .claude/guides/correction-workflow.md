# Correction Workflow

**Version:** 1.1.0
**Last Updated:** 2025-01-20
**Used by:** /proof, /fact-check

Shared workflow for applying corrections from proof and fact-check reports using git worktrees as the safety mechanism.

## Overview

After `/proof` or `/fact-check` generates a corrections report, this workflow:

1. Creates a worktree to isolate changes
2. Applies corrections automatically
3. Shows changes via git diff
4. Lets user approve, reject, or keep for review

**Git is the undo mechanism** - no custom rollback code needed.

---

## When to Use

This workflow is called by `/proof` and `/fact-check` after they generate corrections. Skip this workflow if:

- `--report-only` flag was passed
- No corrections were found
- Target file is read-only (PDF, external URL)

---

## Step 1: Create Worktree

```bash
# Get repo name and target file
REPO=$(basename $(git rev-parse --show-toplevel))
TARGET_FILE="$1"  # e.g., chapters/intro.md
WORKTREE_NAME="proof-$(date +%Y%m%d-%H%M%S)"

# Ensure main is current
cd ~/Documents/GitHub/$REPO
git checkout main && git pull origin main

# Create worktree
mkdir -p ../${REPO}-worktrees
git worktree add ../${REPO}-worktrees/$WORKTREE_NAME -b fix/$WORKTREE_NAME
cd ../${REPO}-worktrees/$WORKTREE_NAME
```

---

## Step 2: Apply Corrections

For each correction in the report, apply using the Edit tool.

### Correction Report Format

Commands must output corrections following the [Correction Report Schema](../templates/correction-report-schema.md). The schema defines:

- **Required fields:** Location, Severity, Context (before), Suggested fix, Rationale
- **Optional fields:** Category, Source, Confidence (for fact-checking)
- **Severity levels:** Error, Warning, Suggestion

### Apply Logic

For each correction:

1. **Read the target file** to verify context exists
2. **Find the exact text** from "Context (before)"
3. **Apply the edit:**

   ```
   Edit tool:
   - file_path: [from Location]
   - old_string: [Context before]
   - new_string: [Suggested fix]
   ```

4. **Log result:** Applied / Skipped (not found) / Failed

### Apply Order

Apply corrections **bottom-to-top** (highest line numbers first) to preserve line numbers for subsequent edits.

### Skip Conditions

Skip a correction if:

- Context text not found in file
- Already matches suggested fix
- Severity is "Suggestion" and user didn't request suggestions

---

## Step 3: Show Changes

After applying all corrections:

```bash
# Show summary
git diff --stat main

# Show actual changes
git diff main
```

Present to user:

```markdown
## Corrections Applied

**Worktree:** fix/proof-20250119-143022
**Target:** chapters/intro.md

| # | Line | Change | Status |
|---|------|--------|--------|
| 1 | 42 | 39% → 36% | Applied |
| 2 | 78 | Added comma | Applied |
| 3 | 156 | Citation format | Skipped (not found) |

### Diff

[Show git diff output]

### Options

- **"approve"** - Commit, merge to main, delete worktree
- **"reject"** - Delete worktree, no changes kept
- **"keep"** - Leave worktree for manual review
```

---

## Step 4: Handle User Decision

Wait for user response.

### If "approve"

```bash
# Commit changes
git add .
git commit -m "$(cat <<'EOF'
fix: Apply corrections from /proof

- [list of changes]

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"

# Merge to main
git checkout main
git merge fix/$WORKTREE_NAME --no-edit

# Push
git push origin main

# Cleanup
cd ~/Documents/GitHub/$REPO
git worktree remove ../${REPO}-worktrees/$WORKTREE_NAME
git branch -d fix/$WORKTREE_NAME
```

Report: "Changes merged to main and pushed."

### If "reject"

```bash
# Just delete worktree
cd ~/Documents/GitHub/$REPO
git worktree remove --force ../${REPO}-worktrees/$WORKTREE_NAME
git branch -D fix/$WORKTREE_NAME
```

Report: "Changes discarded. No modifications made to main."

### If "keep"

```bash
# Leave worktree in place
echo "Worktree kept at: ../${REPO}-worktrees/$WORKTREE_NAME"
echo "To resume: cd ../${REPO}-worktrees/$WORKTREE_NAME"
echo "To approve later: git checkout main && git merge fix/$WORKTREE_NAME"
echo "To discard: git worktree remove ../${REPO}-worktrees/$WORKTREE_NAME"
```

Report: "Worktree preserved for manual review."

---

## Flags

Commands using this workflow should support:

| Flag | Effect |
|------|--------|
| `--report-only` | Generate report but don't apply corrections |
| `--auto-approve` | Apply and merge without prompting (use carefully) |
| `--include-suggestions` | Apply suggestions, not just errors/warnings |

---

## Error Handling

### Worktree creation fails

```bash
# Branch already exists
git branch -D fix/$WORKTREE_NAME 2>/dev/null
git worktree prune
# Retry creation
```

### No corrections found

Skip workflow, report: "No corrections needed."

### All corrections skipped

Report which corrections couldn't be applied and why. Don't create empty commit.

### Merge conflict

```bash
# Should not happen (worktree is fresh from main)
# If it does, report error and keep worktree for manual resolution
```

---

## Integration Example

In `/proof` command:

```markdown
### After generating corrections

If corrections were found and `--report-only` was NOT passed:

1. Follow the workflow in [correction-workflow.md](../guides/correction-workflow.md)
2. Pass the corrections list and target file to the workflow
3. Handle user decision as described

If `--report-only` was passed:
- Present report only
- Suggest: "Run without --report-only to apply corrections"
```

---

## Version History

### 1.1.0 (2025-01-20)

- Standardized terminology: "Phase" → "Step"

### 1.0.0 (2025-01-19)

- Initial release
- Worktree-based correction workflow
- Integration with /proof and /fact-check commands

---

## Related

- [Correction Report Schema](../templates/correction-report-schema.md) - Standard format for corrections
- [Autonomous Cycle](autonomous-cycle.md) - Full dev cycle with worktrees
- `/proof` - Generates proofreading corrections
- `/fact-check` - Generates fact-checking corrections

---

## Version History

### 1.1.0 (2025-01-20)

- Replaced inline format specification with reference to correction-report-schema.md

### 1.0.0 (2025-01-19)

- Initial release
