# Document Proofreading Command

**Version:** 3.4.0
**Last Updated:** 2025-01-20

Proofread a document, generate corrections, and optionally apply them via git worktree.

## Usage

| Command | Action |
|---------|--------|
| `/proof path/to/file.md` | Proof and apply corrections |
| `/proof path/to/file.md --report-only` | Generate report without applying |
| `/proof path/to/file.pdf` | Proof PDF (report only, can't edit) |

## Supported Formats

| Format | Can Edit | Notes |
|--------|----------|-------|
| Markdown (.md) | Yes | Full workflow |
| Plain text (.txt) | Yes | Full workflow |
| Quarto (.qmd) | Yes | Full workflow |
| LaTeX (.tex) | Yes | Full workflow |
| PDF (.pdf) | No | Report only |

---

## Instructions

You are a professional proofreader. Review the document systematically, generate a corrections report, and apply fixes.

---

### Step 1: Setup

1. **Parse arguments:**
   - Extract file path from `$ARGUMENTS`
   - Check for `--report-only` flag
   - Determine if file is editable (not PDF)

2. **Check for local style guides** - Review CLAUDE.md for any linked style guides and apply those conventions. See [CONFIGURATION.md](../CONFIGURATION.md) for style guide configuration options.

3. **Identify document type** from file extension

4. **Get document info:**

   **For PDF:**

   ```bash
   python -c "from pypdf import PdfReader; r=PdfReader('$FILE'); print(f'Pages: {len(r.pages)}')"
   ```

   **For Markdown/Text:**

   ```bash
   wc -l "$FILE"
   ```

5. **Plan review strategy** based on size:
   - < 500 lines: Review all
   - 500-2000 lines: Review in batches
   - > 2000 lines: Focus on key sections

---

### Step 2: Document Review

#### For PDF Documents

Extract and review page by page:

```bash
uv run python scripts/pdf_page_extractor.py "$FILE" --page [N] --text
```

#### For Markdown/Text Documents

Read in chunks using the Read tool with offset and limit parameters.

#### What to Check

**Typography:**

- Orphans/widows
- Bad line/word breaks
- Inconsistent spacing
- Missing/incorrect punctuation

**Content:**

- Typos and spelling errors
- Grammar issues
- Broken cross-references ("Figure ??")
- Incorrect numbering
- Inconsistent terminology

**Markdown-Specific:**

- Broken links `[text]()`
- Inconsistent heading levels
- Unclosed code blocks
- Malformed lists

**Academic/Technical:**

- Citation format consistency
- Missing references
- Equation formatting

#### Search for Common Problems

```bash
# Broken links
grep -n "\[.*\]()" "$FILE"

# TODOs
grep -n "TODO\|FIXME\|XXX" "$FILE"

# Double spaces
grep -n "  " "$FILE"

# Broken references
grep -n "\?\?" "$FILE"
```

---

### Step 3: Generate Corrections Report

Format corrections following the [Correction Report Schema](../templates/correction-report-schema.md). The report should include:

1. **Header** with document name, timestamp, target file path, and editability
2. **Summary table** with counts by severity (Errors, Warnings, Suggestions)
3. **Corrections section** with each item containing:
   - Numbered description (e.g., `[1] Typo: "teh" should be "the"`)
   - Location (`path/to/file.md:line`)
   - Severity (Error, Warning, or Suggestion)
   - Context (before) with 2-3 surrounding lines
   - Suggested fix with same context
   - Rationale for the change
4. **Items Not Auto-Fixable** table for issues requiring author decision

**Important:** Include 2-3 lines of context in "Context (before)" to ensure unique matching.

---

### Step 4: Apply Corrections

**If `--report-only` OR file is PDF:**

- Present report only
- Suggest: "Run without --report-only to apply corrections" (if applicable)
- Stop here

**If editable file and corrections found:**
Follow the [correction-workflow](../guides/correction-workflow.md) guide.

**If no corrections found:**

- Report: "No corrections needed. Document looks good!"

---

### Step 5: Report Results

After user decision (or for report-only):

```markdown
## Proof Complete

**Document:** [filename]
**Corrections:** X applied, Y skipped
**Status:** [Merged to main / Rejected / Kept for review]

### Changes Made
- Line 42: Fixed typo "teh" → "the"
- Line 78: Added Oxford comma

### Skipped
- Line 156: Unclear reference (requires author decision)
```

---

## Flags

| Flag | Effect |
|------|--------|
| `--report-only` | Generate report without applying corrections |
| `--auto-approve` | Apply and merge without prompting |
| `--include-suggestions` | Apply suggestions, not just errors/warnings |

---

## Important Notes

- **One section at a time** to avoid context overflow
- **Focus on actionable issues** - don't nitpick style unless asked
- **Prioritize errors** over warnings over suggestions
- **Include context** - 2-3 lines around each issue for matching
- **PDF is report-only** - can't edit PDF files

---

## Troubleshooting

### Context not found when applying correction

- The document may have changed since the report was generated
- Try regenerating the report with `/proof --report-only`
- Ensure context includes enough surrounding lines for unique matching

### Worktree creation fails

```bash
git worktree prune  # Clean up stale worktrees
git worktree list   # Check existing worktrees
```

### Corrections applied incorrectly

- Use `git diff` in the worktree to review changes
- Choose "reject" to discard and try again
- Report may need more context for unique matching

---

## Related

- `/fact-check` - Verify empirical claims (different purpose)
- [Correction Report Schema](../templates/correction-report-schema.md) - Standard format for corrections
- [CONFIGURATION.md](../CONFIGURATION.md) - All CLAUDE.md configuration options
- [correction-workflow](../guides/correction-workflow.md) - Shared apply logic
- [lint-and-hooks](../guides/lint-and-hooks.md) - Pre-commit hook handling

---

## Version History

### 3.4.0 (2025-01-20)

- Replaced inline format specification with reference to correction-report-schema.md

### 3.3.0 (2025-01-20)

- Added reference to CONFIGURATION.md for style guide options
- Added CONFIGURATION.md to Related section

### 3.2.0 (2025-01-20)

- Added Troubleshooting section
- Added Related section

### 3.1.0 (2025-01-20)

- Standardized terminology: "Phase" -> "Step"
- Simplified correction workflow reference (removed inline duplication)

### 3.0.0 (2025-01-19)

- Added worktree-based correction workflow
- Auto-applies corrections with git as undo mechanism
- Added --report-only flag
- Standardized correction report format
- Integrated with correction-workflow.md guide

### 2.1.0 (2025-01-19)

- Added step to check CLAUDE.md for linked style guides

### 2.0.0 (2025-01-19)

- Generalized from chirho PDF proofer
- Added support for Markdown and plain text

### 1.0.0

- Initial PDF-specific implementation
