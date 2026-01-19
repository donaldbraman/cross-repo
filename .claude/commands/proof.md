# Document Proofreading Command

**Version:** 2.1.0
**Last Updated:** 2025-01-19

Proofread a document for errors, inconsistencies, and issues. Use `$ARGUMENTS` to specify the file path and optional parameters.

## Supported Formats

- **PDF**: Page-by-page visual review
- **Markdown**: Content and formatting review
- **Plain text**: Content review
- **LaTeX**: Source and compiled review

## Instructions

You are a professional proofreader. Your task is to systematically review a document for errors, inconsistencies, and issues.

### Setup

1. **Check for local style guides** - Review CLAUDE.md for any linked style guides and apply those conventions when proofreading.

2. **Identify document type** from the file extension
3. **Get document info** to understand structure:

**For PDF:**
```bash
# If pdf_page_extractor.py exists:
uv run python scripts/pdf_page_extractor.py "$ARGUMENTS" --info

# Otherwise use pypdf directly:
python -c "from pypdf import PdfReader; r=PdfReader('$ARGUMENTS'); print(f'Pages: {len(r.pages)}')"
```

**For Markdown/Text:**
```bash
wc -l "$ARGUMENTS"
head -50 "$ARGUMENTS"
```

4. **Plan review strategy** based on document size:
   - < 20 pages/500 lines: Review all
   - 20-50 pages/500-2000 lines: Review in batches
   - > 50 pages/2000 lines: Focus on key sections

### Document Review

#### For PDF Documents

Extract and review one page at a time:

```bash
# Extract page as image
uv run python scripts/pdf_page_extractor.py "$ARGUMENTS" --page [N] --output /tmp/proof_page.png

# Or extract text
uv run python scripts/pdf_page_extractor.py "$ARGUMENTS" --page [N] --text
```

Then read the extracted page image or text.

#### For Markdown/Text Documents

Read the document in chunks using the Read tool with offset and limit parameters.

### What to Check

**Typography:**
- Orphans (single lines at top of page)
- Widows (single lines at bottom of page)
- Bad line/word breaks
- Inconsistent spacing
- Missing or incorrect punctuation

**Layout (PDF/LaTeX):**
- Figures/tables in wrong position
- Missing captions
- Text overflow or cutoff
- Misaligned elements
- Inconsistent margins

**Content:**
- Typos and spelling errors
- Grammar issues
- Broken cross-references (e.g., "Figure ??" or "Table ??")
- Incorrect numbering
- Inconsistent terminology

**Academic/Technical:**
- Equation formatting issues
- Citation format consistency
- Bibliography completeness
- Missing references
- Incorrect figure/table references

**Markdown-Specific:**
- Broken links
- Inconsistent heading levels
- Unclosed code blocks
- Malformed lists
- Missing alt text for images

### Search for Common Problems

**For PDF:**
```bash
# Find broken references
uv run python scripts/pdf_page_extractor.py "$ARGUMENTS" --search "\?\?"

# Find figure/table references
uv run python scripts/pdf_page_extractor.py "$ARGUMENTS" --search "Figure \d+"
uv run python scripts/pdf_page_extractor.py "$ARGUMENTS" --search "Table \d+"
```

**For Markdown/Text:**
```bash
# Find broken links
grep -n "\[.*\]()" "$ARGUMENTS"

# Find TODO/FIXME markers
grep -n "TODO\|FIXME\|XXX" "$ARGUMENTS"

# Find double spaces
grep -n "  " "$ARGUMENTS"
```

### Reporting

After reviewing, provide a structured report:

```markdown
## Proofreading Report: [Filename]

### Summary
- Content reviewed: X pages/lines of Y
- Critical issues: N
- Minor issues: N
- Suggestions: N

### Critical Issues (Must Fix)
1. Page/Line X: [Description]
2. Page/Line Y: [Description]

### Minor Issues (Should Fix)
1. Page/Line X: [Description]

### Suggestions (Optional)
1. Page/Line X: [Suggestion]

### Review Progress
- [x] Pages/Lines 1-10
- [ ] Pages/Lines 11-20
- ...
```

### Workflow

1. Start with document info to understand structure
2. Search for obvious problems (broken refs, TODOs, etc.)
3. Review title/intro and key sections visually
4. For thorough review, go page by page or section by section
5. Compile and present report

### Important Notes

- **One page/section at a time** to avoid context overflow
- **Focus on actionable issues** - don't nitpick style unless asked
- **Prioritize critical issues** - broken references, missing content
- **Be consistent** - use the same reporting format throughout

---

## Version History

### 2.1.0 (2025-01-19)
- Added step to check CLAUDE.md for linked style guides before proofreading

### 2.0.0 (2025-01-19)
- Generalized from chirho PDF proofer
- Added support for Markdown and plain text
- Made script paths configurable
- Added search patterns for different formats

### 1.0.0
- Initial PDF-specific implementation (chirho)
