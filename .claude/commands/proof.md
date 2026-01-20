# Document Proofreading Command

**Version:** 3.0.0
**Last Updated:** 2025-01-19

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

### Phase 1: Setup

1. **Parse arguments:**
   - Extract file path from `$ARGUMENTS`
   - Check for `--report-only` flag
   - Determine if file is editable (not PDF)

2. **Check for local style guides** - Review CLAUDE.md for any linked style guides and apply those conventions.

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

### Phase 2: Document Review

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

### Phase 3: Generate Corrections Report

Format corrections for the [correction-workflow](../guides/correction-workflow.md):

```markdown
## Proofreading Report: [Filename]

**Generated:** [timestamp]
**Target:** [file path]
**Editable:** Yes/No

### Summary

| Category | Count |
|----------|-------|
| Errors | X |
| Warnings | X |
| Suggestions | X |

---

### Corrections

#### [1] Typo: "teh" → "the"

- **Location:** `path/to/file.md:42`
- **Severity:** Error
- **Context (before):**
  ```
  This is teh example
  text that needs fixing.
  ```
- **Suggested fix:**
  ```
  This is the example
  text that needs fixing.
  ```
- **Reason:** Spelling error

#### [2] Missing comma before "and"

- **Location:** `path/to/file.md:78`
- **Severity:** Warning
- **Context (before):**
  ```
  apples, oranges and bananas
  ```
- **Suggested fix:**
  ```
  apples, oranges, and bananas
  ```
- **Reason:** Oxford comma for consistency

---

### Items Not Auto-Fixable

| Line | Issue | Reason |
|------|-------|--------|
| 156 | Unclear reference | Requires author decision |
| 203 | Missing citation | Source unknown |
```

**Important:** Include 2-3 lines of context in "Context (before)" to ensure unique matching.

---

### Phase 4: Apply Corrections

**If `--report-only` OR file is PDF:**
- Present report only
- Suggest: "Run without --report-only to apply corrections" (if applicable)
- Stop here

**If editable file and corrections found:**

Follow the [correction-workflow](../guides/correction-workflow.md):

1. Create worktree to isolate changes
2. Apply each correction using Edit tool
3. Show git diff of changes
4. Present options: approve / reject / keep

**If no corrections found:**
- Report: "No corrections needed. Document looks good!"

---

### Phase 5: Report Results

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

## Version History

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
