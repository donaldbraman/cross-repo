# Worked Example: Correction Workflow

**Version:** 1.0.0
**Last Updated:** 2025-01-20
**Related:** [/proof](../commands/proof.md), [correction-workflow guide](../guides/correction-workflow.md)

Complete walkthrough of running `/proof` on a document and applying corrections.

---

## Scenario

**Task:** Proofread a Markdown chapter file and apply corrections.

**Repository:** `research-paper` (hypothetical)
**Target file:** `chapters/methodology.md`
**Starting point:** User runs `/proof chapters/methodology.md`

---

## Step 1: Setup

The agent parses arguments and prepares for review.

```bash
# Agent identifies the file
$ ls -la chapters/methodology.md
-rw-r--r--  1 user  staff  4523 Jan 20 09:00 chapters/methodology.md

# Agent counts lines to plan review strategy
$ wc -l chapters/methodology.md
     156 chapters/methodology.md
```

**Agent output:**
> File has 156 lines. Will review the entire document in one pass.
> Checking for style guides in CLAUDE.md... None found.
> Document type: Markdown

---

## Step 2: Document Review

The agent reads the file and identifies issues.

**Sample content from `chapters/methodology.md`:**

```markdown
## Methodology

### 2.1 Partipant Selection

We recruited 120 participants form a univeristy pool. Participants
were between the ages of 18-65 and had no prior exposure to teh
experimental stimulus.

### 2.2 Procedure

Each partipant completed a series of tasks:

1. Pre-test questionaire
2. Exposure to stimulus (15 minutes)
3. Post-test questionare
4. Follow-up interview (optional)

The error rate was 39% higher than the control group, which
suggests a significant effect (p < 0.05).

### 2.3 Analysis

Data was analyzed using SPSS version 27.  We applied a
mixed-effects model with participant as a random effect.
```

### Issues Found

The agent identifies the following problems:

| Line | Issue | Severity |
|------|-------|----------|
| 3 | "Partipant" should be "Participant" | Error |
| 5 | "form" should be "from" | Error |
| 5 | "univeristy" should be "university" | Error |
| 6-7 | "teh" should be "the" | Error |
| 11 | "partipant" should be "participant" | Error |
| 13 | "questionaire" should be "questionnaire" | Error |
| 15 | "questionare" should be "questionnaire" | Error |
| 21 | Double space after "27." | Warning |

---

## Step 3: Generate Corrections Report

The agent generates a structured report following the [correction report schema](../templates/correction-report-schema.md).

```markdown
## Proofreading Report: methodology.md

**Source:** /proof
**Generated:** 2025-01-20 10:30:00
**Target:** chapters/methodology.md
**Editable:** Yes

### Summary

| Category | Count |
|----------|-------|
| Errors | 7 |
| Warnings | 1 |
| Suggestions | 0 |

---

### Corrections

#### [1] Typo: "Partipant" should be "Participant"

- **Location:** `chapters/methodology.md:3`
- **Severity:** Error
- **Context (before):**
  ```
  ## Methodology

  ### 2.1 Partipant Selection
  ```
- **Suggested fix:**
  ```
  ## Methodology

  ### 2.1 Participant Selection
  ```
- **Rationale:** Spelling error in section heading

#### [2] Typo: "form" should be "from"

- **Location:** `chapters/methodology.md:5`
- **Severity:** Error
- **Context (before):**
  ```
  ### 2.1 Participant Selection

  We recruited 120 participants form a univeristy pool. Participants
  ```
- **Suggested fix:**
  ```
  ### 2.1 Participant Selection

  We recruited 120 participants from a univeristy pool. Participants
  ```
- **Rationale:** Common typo - "form" vs "from"

#### [3] Typo: "univeristy" should be "university"

- **Location:** `chapters/methodology.md:5`
- **Severity:** Error
- **Context (before):**
  ```
  We recruited 120 participants from a univeristy pool. Participants
  were between the ages of 18-65 and had no prior exposure to teh
  ```
- **Suggested fix:**
  ```
  We recruited 120 participants from a university pool. Participants
  were between the ages of 18-65 and had no prior exposure to teh
  ```
- **Rationale:** Spelling error

#### [4] Typo: "teh" should be "the"

- **Location:** `chapters/methodology.md:6`
- **Severity:** Error
- **Context (before):**
  ```
  were between the ages of 18-65 and had no prior exposure to teh
  experimental stimulus.
  ```
- **Suggested fix:**
  ```
  were between the ages of 18-65 and had no prior exposure to the
  experimental stimulus.
  ```
- **Rationale:** Common typo

#### [5] Typo: "partipant" should be "participant"

- **Location:** `chapters/methodology.md:11`
- **Severity:** Error
- **Context (before):**
  ```
  ### 2.2 Procedure

  Each partipant completed a series of tasks:
  ```
- **Suggested fix:**
  ```
  ### 2.2 Procedure

  Each participant completed a series of tasks:
  ```
- **Rationale:** Spelling error

#### [6] Typo: "questionaire" should be "questionnaire"

- **Location:** `chapters/methodology.md:13`
- **Severity:** Error
- **Context (before):**
  ```
  Each participant completed a series of tasks:

  1. Pre-test questionaire
  2. Exposure to stimulus (15 minutes)
  ```
- **Suggested fix:**
  ```
  Each participant completed a series of tasks:

  1. Pre-test questionnaire
  2. Exposure to stimulus (15 minutes)
  ```
- **Rationale:** Spelling error

#### [7] Typo: "questionare" should be "questionnaire"

- **Location:** `chapters/methodology.md:15`
- **Severity:** Error
- **Context (before):**
  ```
  2. Exposure to stimulus (15 minutes)
  3. Post-test questionare
  4. Follow-up interview (optional)
  ```
- **Suggested fix:**
  ```
  2. Exposure to stimulus (15 minutes)
  3. Post-test questionnaire
  4. Follow-up interview (optional)
  ```
- **Rationale:** Spelling error

#### [8] Extra space: double space after period

- **Location:** `chapters/methodology.md:21`
- **Severity:** Warning
- **Context (before):**
  ```
  ### 2.3 Analysis

  Data was analyzed using SPSS version 27.  We applied a
  mixed-effects model with participant as a random effect.
  ```
- **Suggested fix:**
  ```
  ### 2.3 Analysis

  Data was analyzed using SPSS version 27. We applied a
  mixed-effects model with participant as a random effect.
  ```
- **Rationale:** Standard style uses single space after period

---

### Items Not Auto-Fixable

| Line | Issue | Reason |
|------|-------|--------|
| 17-18 | "39% higher" | Verify against source data (may need fact-check) |
```

---

## Step 4: Apply Corrections

Since the file is editable and corrections were found, the agent follows the [correction-workflow](../guides/correction-workflow.md).

### Create Worktree

```bash
$ REPO=$(basename $(git rev-parse --show-toplevel))
$ echo $REPO
research-paper

$ WORKTREE_NAME="proof-20250120-103000"

$ cd ~/Documents/GitHub/$REPO
$ git checkout main && git pull origin main
Already on 'main'
Already up to date.

$ mkdir -p ../${REPO}-worktrees
$ git worktree add ../${REPO}-worktrees/$WORKTREE_NAME -b fix/$WORKTREE_NAME
Preparing worktree (new branch 'fix/proof-20250120-103000')
HEAD is now at abc1234 docs: Update introduction

$ cd ../${REPO}-worktrees/$WORKTREE_NAME
$ pwd
/Users/user/Documents/GitHub/research-paper-worktrees/proof-20250120-103000
```

### Apply Edits

The agent applies corrections **bottom-to-top** to preserve line numbers:

```
Applying correction [8] at line 21... Applied
Applying correction [7] at line 15... Applied
Applying correction [6] at line 13... Applied
Applying correction [5] at line 11... Applied
Applying correction [4] at line 6... Applied
Applying correction [3] at line 5... Applied
Applying correction [2] at line 5... Applied
Applying correction [1] at line 3... Applied
```

### Show Changes

```bash
$ git diff --stat main
 chapters/methodology.md | 14 +++++++-------
 1 file changed, 7 insertions(+), 7 deletions(-)

$ git diff main
diff --git a/chapters/methodology.md b/chapters/methodology.md
index abc1234..def5678 100644
--- a/chapters/methodology.md
+++ b/chapters/methodology.md
@@ -1,19 +1,19 @@
 ## Methodology

-### 2.1 Partipant Selection
+### 2.1 Participant Selection

-We recruited 120 participants form a univeristy pool. Participants
-were between the ages of 18-65 and had no prior exposure to teh
+We recruited 120 participants from a university pool. Participants
+were between the ages of 18-65 and had no prior exposure to the
 experimental stimulus.

 ### 2.2 Procedure

-Each partipant completed a series of tasks:
+Each participant completed a series of tasks:

-1. Pre-test questionaire
+1. Pre-test questionnaire
 2. Exposure to stimulus (15 minutes)
-3. Post-test questionare
+3. Post-test questionnaire
 4. Follow-up interview (optional)

 The error rate was 39% higher than the control group, which
@@ -18,7 +18,7 @@ suggests a significant effect (p < 0.05).

 ### 2.3 Analysis

-Data was analyzed using SPSS version 27.  We applied a
+Data was analyzed using SPSS version 27. We applied a
 mixed-effects model with participant as a random effect.
```

### Present to User

```markdown
## Corrections Applied

**Worktree:** fix/proof-20250120-103000
**Target:** chapters/methodology.md

| # | Line | Change | Status |
|---|------|--------|--------|
| 1 | 3 | Partipant -> Participant | Applied |
| 2 | 5 | form -> from | Applied |
| 3 | 5 | univeristy -> university | Applied |
| 4 | 6 | teh -> the | Applied |
| 5 | 11 | partipant -> participant | Applied |
| 6 | 13 | questionaire -> questionnaire | Applied |
| 7 | 15 | questionare -> questionnaire | Applied |
| 8 | 21 | double space -> single space | Applied |

### Note
Line 17-18: The statistic "39% higher" should be verified against source data.
Consider running `/fact-check chapters/methodology.md` for verification.

### Options

- **"approve"** - Commit, merge to main, delete worktree
- **"reject"** - Delete worktree, no changes kept
- **"keep"** - Leave worktree for manual review
```

---

## User Decision: Approve

The user types "approve".

### Commit Changes

```bash
$ git add .
$ git commit -m "$(cat <<'EOF'
fix: Apply corrections from /proof

- Fixed 7 spelling errors (Partipant, form, univeristy, teh, partipant, questionaire, questionare)
- Fixed 1 formatting issue (double space after period)

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
[fix/proof-20250120-103000 def5678] fix: Apply corrections from /proof
 1 file changed, 7 insertions(+), 7 deletions(-)
```

### Merge to Main

```bash
$ git checkout main
Switched to branch 'main'

$ git merge fix/proof-20250120-103000 --no-edit
Updating abc1234..def5678
Fast-forward
 chapters/methodology.md | 14 +++++++-------
 1 file changed, 7 insertions(+), 7 deletions(-)

$ git push origin main
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 10 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 423 bytes | 423.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0)
To https://github.com/user/research-paper.git
   abc1234..def5678  main -> main
```

### Cleanup

```bash
$ cd ~/Documents/GitHub/research-paper
$ git worktree remove ../research-paper-worktrees/proof-20250120-103000
$ git branch -d fix/proof-20250120-103000
Deleted branch fix/proof-20250120-103000 (was def5678).
```

### Final Report

```markdown
## Proof Complete

**Document:** chapters/methodology.md
**Corrections:** 8 applied, 0 skipped
**Status:** Merged to main

### Changes Made
- Line 3: Fixed "Partipant" -> "Participant"
- Line 5: Fixed "form" -> "from"
- Line 5: Fixed "univeristy" -> "university"
- Line 6: Fixed "teh" -> "the"
- Line 11: Fixed "partipant" -> "participant"
- Line 13: Fixed "questionaire" -> "questionnaire"
- Line 15: Fixed "questionare" -> "questionnaire"
- Line 21: Fixed double space after period

### Recommendation
Run `/fact-check chapters/methodology.md` to verify the "39% higher" statistic.
```

---

## Alternative: User Rejects

If the user types "reject":

```bash
$ cd ~/Documents/GitHub/research-paper
$ git worktree remove --force ../research-paper-worktrees/proof-20250120-103000
$ git branch -D fix/proof-20250120-103000
Deleted branch fix/proof-20250120-103000 (was def5678).
```

**Report:**
```
Changes discarded. No modifications made to main.
```

---

## Alternative: User Keeps for Review

If the user types "keep":

```markdown
Worktree preserved for manual review.

**Location:** ../research-paper-worktrees/proof-20250120-103000

To resume:
  cd ../research-paper-worktrees/proof-20250120-103000

To approve later:
  git checkout main && git merge fix/proof-20250120-103000

To discard:
  git worktree remove ../research-paper-worktrees/proof-20250120-103000
```

---

## What Success Looks Like

After a successful `/proof` cycle with "approve":

1. **All corrections applied** - Spelling and formatting errors fixed
2. **Changes on main** - Latest main branch contains the fixes
3. **Worktree cleaned up** - Temporary worktree removed
4. **Document improved** - File passes proofreading
5. **Actionable recommendations** - User informed about items needing manual review

---

## Common Variations

### Report-Only Mode

```bash
$ /proof chapters/methodology.md --report-only
```

The agent generates the report but does not create a worktree or apply corrections:

```markdown
## Proofreading Report: methodology.md
[... full report ...]

---
Run without --report-only to apply corrections.
```

### PDF Documents

```bash
$ /proof paper.pdf
```

PDFs are read-only, so the workflow stops after the report:

```markdown
## Proofreading Report: paper.pdf

**Source:** /proof
**Generated:** 2025-01-20 10:30:00
**Target:** paper.pdf
**Editable:** No (PDF files cannot be edited directly)

[... corrections listed but not applied ...]

---
Note: PDF files cannot be edited. Export to a source format to apply corrections.
```

### Including Suggestions

```bash
$ /proof chapters/methodology.md --include-suggestions
```

The agent also applies severity=Suggestion items, not just Error and Warning.

### Auto-Approve Mode

```bash
$ /proof chapters/methodology.md --auto-approve
```

The agent applies corrections and merges without prompting. Use with caution.

---

## Edge Cases

### Context Not Found

If the document changed since the report was generated:

```
Applying correction [3] at line 5... Skipped (context not found)
```

The correction is logged as skipped. The agent continues with other corrections.

### No Corrections Found

```markdown
## Proof Complete

**Document:** chapters/conclusion.md
**Status:** No corrections needed. Document looks good!
```

No worktree is created if there's nothing to fix.

### All Corrections Already Applied

If you run `/proof` twice on the same file:

```
Applying correction [1] at line 3... Skipped (already correct)
Applying correction [2] at line 5... Skipped (already correct)
[...]

All corrections were already applied or not applicable.
No changes made.
```

### Large Document

For documents over 2000 lines:

```markdown
**Planning review strategy:**
Document has 3,450 lines. Will review in sections:
- Section 1 (Introduction): lines 1-500
- Section 2 (Literature Review): lines 501-1200
- Section 3 (Methodology): lines 1201-1800
- Section 4 (Results): lines 1801-2800
- Section 5 (Discussion): lines 2801-3450

Reviewing Section 1...
[...]
```

---

## Related Examples

- [autonomous-cycle-example.md](autonomous-cycle-example.md) - Full development cycle with issue, worktree, PR, merge
