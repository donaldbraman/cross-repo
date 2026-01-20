# Fact-Check Command

**Version:** 3.0.0
**Last Updated:** 2025-01-20

Verify empirical claims against original sources, generate corrections, and optionally apply them via git worktree.

## Quick Reference

| Command | Action |
|---------|--------|
| `/fact-check path/to/file.md` | Fact-check and apply corrections |
| `/fact-check path/to/file.md --report-only` | Generate report without applying |
| `/fact-check --section "Results"` | Fact-check specific section only |
| `/fact-check --sequential` | Disable parallel execution |

## What This Command Verifies

Unlike `/proof` (which checks formatting and style), `/fact-check` verifies **empirical accuracy**:

1. **Statistics** - Do numbers match original sources?
2. **Study findings** - Are findings characterized accurately?
3. **Citations** - Do sources exist with correct author/year/publication?
4. **URLs** - Do links work and point to correct content?
5. **Domain-specific claims** - Legal holdings, technical specs, etc. (project-configurable)

---

## Instructions

When the user runs `/fact-check [file]`, execute the following steps. For detailed methodology, see [fact-check-methodology.md](fact-check-methodology.md).

### Step Overview

| Step | Description | Details |
|------|-------------|---------|
| 0 | Configuration | Check CLAUDE.md for project settings |
| 1 | Extract Claims | Identify verifiable claims in document |
| 2 | Prioritize | Rank claims by importance |
| 3 | Decompose | Break compound claims into atomic facts |
| 4 | Verify | Search sources using tiered approach |
| 5 | Parallel Execution | Spawn agents for large documents |
| 6 | Generate Report | Create corrections report |
| 7 | Apply Corrections | Follow correction-workflow guide |
| 8 | Report Results | Summarize changes made |

### Quick Execution Guide

1. **Extract claims** from document (high priority: statistics, study findings, citations)
2. **Verify each claim** using tiered search: local sources, external databases, web search
3. **Assign verdicts**: SUPPORTED, PARTIALLY SUPPORTED, UNSUPPORTED, or CONTRADICTED
4. **Generate report** in [Correction Report Schema](../templates/correction-report-schema.md) format
5. **Apply corrections** via [correction-workflow](../guides/correction-workflow.md) unless `--report-only`

For claim types and verification patterns, see [fact-check-reference.md](fact-check-reference.md).

---

## Flags

| Flag | Effect |
|------|--------|
| `--report-only` | Generate report without applying corrections |
| `--sequential` | Disable parallel execution |
| `--auto-approve` | Apply and merge without prompting |
| `--section "X"` | Only fact-check named section |

---

## Project Configuration

Projects can customize fact-checking by adding configuration to their CLAUDE.md. See [CONFIGURATION.md](../CONFIGURATION.md) for all available options, detailed descriptions, and examples.

---

## Troubleshooting

### Source not found
- Try different search terms or author name variations
- Check if URL is behind paywall or requires login
- Use `--report-only` to flag for manual verification

### Parallel agents timing out
- Use `--sequential` flag to disable parallelization
- Break document into smaller sections with `--section`

### Context not found when applying correction
- Document may have changed since report was generated
- Regenerate report with `/fact-check --report-only`
- Ensure context includes enough lines for unique matching

### Worktree creation fails
```bash
git worktree prune  # Clean up stale worktrees
git worktree list   # Check existing worktrees
```

---

## Version History

### 3.0.0 (2025-01-20)
- Split into modular files for maintainability
- Created fact-check-methodology.md with detailed steps
- Created fact-check-reference.md with claim types and red flags
- Reduced main file from ~500 to ~150 lines

### 2.4.0 (2025-01-20)
- Replaced inline format specification with reference to correction-report-schema.md

### 2.3.0 (2025-01-20)
- Extracted configuration examples to central CONFIGURATION.md
- Added reference to CONFIGURATION.md in Related section

### 2.2.0 (2025-01-20)
- Added Troubleshooting section

### 2.1.0 (2025-01-20)
- Standardized terminology: "Phase" -> "Step"
- Simplified correction workflow reference (removed inline duplication)
- Renamed sub-steps in Step 5 to "Part A-E" to avoid confusion

### 2.0.0 (2025-01-19)
- Added worktree-based correction workflow
- Auto-applies corrections with git as undo mechanism
- Added --report-only flag
- Integrated with correction-workflow.md guide
- Added Step 8 for results reporting

### 1.0.0 (2025-01-19)
- Initial cross-repo release
- Generalized from criminal-law project-specific version
- Made verification tools configurable via CLAUDE.md

---

## Related

- [fact-check-methodology](fact-check-methodology.md) - Detailed step-by-step process
- [fact-check-reference](fact-check-reference.md) - Claim types, red flags, patterns
- `/proof` - Formatting and style checks (different purpose)
- [Correction Report Schema](../templates/correction-report-schema.md) - Standard format for corrections
- [CONFIGURATION.md](../CONFIGURATION.md) - All CLAUDE.md configuration options
- [correction-workflow](../guides/correction-workflow.md) - Shared apply logic
