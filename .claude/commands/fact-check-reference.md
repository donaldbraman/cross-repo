# Fact-Check Reference

**Version:** 1.0.0
**Last Updated:** 2025-01-20

Reference guide for claim types, red flags, and verification patterns. This is a companion file to the main [fact-check.md](fact-check.md) command.

---

## Claim Type Reference

### Statistics

```
Check:
- Exact number matches
- Percentage calculated correctly
- Time period accurate
- Sample/population correct

Common errors:
- Transposed digits (26% vs 62%)
- Wrong year's data
- Median vs mean confusion
```

### Study Findings

```
Check:
- Study exists, correctly cited
- Finding accurately characterized
- Effect size/direction correct
- Significance not overstated

Common errors:
- "Proves" vs "suggests"
- Missing confidence intervals
- Conflating correlation/causation
```

### Citations

```
Check:
- Author name spelling
- Publication year
- Journal/book title
- URL validity

Common errors:
- Year off by one
- Misspelled names
- Wrong journal
```

---

## Red Flags

| Red Flag | Action |
|----------|--------|
| Round numbers (50%, 100%) | Verify exact figure |
| Superlatives ("largest", "first") | Verify strong claims |
| Causal language ("causes") | Check if source claims causation |
| Missing citations | Flag for source |
| Old dates | Check for updates |

---

## Verification Verdicts

| Verdict | Definition | Action |
|---------|------------|--------|
| **SUPPORTED** | Matches source exactly or within rounding | No correction needed |
| **PARTIALLY SUPPORTED** | Directionally correct but imprecise | Flag as warning |
| **UNSUPPORTED** | Cannot find evidence | Flag for manual review |
| **CONTRADICTED** | Source says something different | Requires correction |

---

## Confidence Levels

| Level | Criteria | Reliability |
|-------|----------|-------------|
| **High** | Accessed original source directly | Very reliable |
| **Medium** | Found corroborating secondary source | Generally reliable |
| **Low** | Only indirect evidence available | Use with caution |
| **Unable** | Could not locate source | Requires manual verification |

---

## Example Output

```markdown
## Fact-Check Report: Chapter 3, Results Section

### Summary
| Verdict | Count |
|---------|-------|
| Supported | 18 |
| Partially Supported | 3 |
| Unsupported | 1 |
| Contradicted | 1 |

**Accuracy Rate:** 78% fully supported

### Errors Requiring Correction

1. **Line 89:** "Effect sizes range from 20-39%"
   - **Issue:** Upper bound incorrect
   - **Source says:** Study found 36%, not 39%
   - **Suggested fix:** "Effect sizes range from 20-36%"

### Warnings

1. **Line 103:** Rate increase "8.9%"
   - **Concern:** Could only find press release, not original data
   - **Confidence:** Medium
```

---

## Priority Calculation

Claims are prioritized for verification using this formula:

```
Priority = (Central to argument) x (Specificity) x (Verifiability)
```

### Target Verification Rates

| Claim Type | Target Rate |
|------------|-------------|
| Statistics and effect sizes | 100% |
| Study characterizations (key arguments) | 100% |
| Citations (author, year, publication) | 80% |
| URLs | 50% (spot-check) |

---

## Common Search Patterns

### Academic Sources

```bash
# Google Scholar
web_search("site:scholar.google.com [author] [year] [topic]")

# Direct PDF search
web_search("[exact title] filetype:pdf")
```

### Government Statistics

```bash
# US government sources
web_search("site:bls.gov OR site:census.gov [topic]")

# Specific agency
web_search("site:[agency].gov [statistic]")
```

### Domain-Specific Sources

Configure additional search domains in CLAUDE.md:

```yaml
## Fact-Check Configuration
fact_check:
  domain_sources:
    - courtlistener.com  # Legal opinions
    - pubmed.gov         # Medical literature
    - arxiv.org          # Physics/CS preprints
```

---

## Related

- [fact-check](fact-check.md) - Main command file
- [fact-check-methodology](fact-check-methodology.md) - Step-by-step process
- [CONFIGURATION.md](../CONFIGURATION.md) - All configuration options
