# Fact-Check Command

**Version:** 2.3.0
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

When the user runs `/fact-check [file]`, execute the following steps:

---

### Step 0: Configuration

1. **Check CLAUDE.md for project-specific settings:**
   - Verification tools (local search APIs, external databases)
   - Domain-specific claim categories
   - Source file locations
   - Report output directory

2. **Check for linked style guides** that may define citation formats or terminology standards.

See [CONFIGURATION.md](../CONFIGURATION.md) for all available options and examples.

If no configuration found, use defaults (web search only, standard categories).

---

### Step 1: Extract Claims

Read the document and identify all verifiable claims:

**High Priority (Always Verify):**
- Statistics with specific numbers (percentages, sample sizes, dollar amounts)
- Study findings ("X found that Y", effect sizes)
- Direct citations with author/year
- Comparative claims ("X is higher/lower than Y")

**Medium Priority (Spot Check):**
- Historical facts and dates
- Domain-specific technical claims
- General characterizations of research

**Skip:**
- Opinions and arguments
- Predictions and hypotheticals
- Quoted material (verify separately with /verify-quotes if available)

Create a list of claims to verify:

```markdown
## Claims Extracted from [Section Name]

1. [Line XX] "A 2023 meta-analysis synthesized twelve RCTs (N=79,255)..."
   - Type: Study finding
   - Atomic facts: (a) 2023 publication, (b) 12 RCTs, (c) N=79,255

2. [Line YY] "Error rates increased 8.9%..."
   - Type: Statistic
   - Source cited: [Source name]
```

---

### Step 2: Prioritize

Rank claims by importance:

```
Priority = (Central to argument) × (Specificity) × (Verifiability)
```

For a typical document:
- Verify 100% of statistics and effect sizes
- Verify 100% of study characterizations supporting key arguments
- Verify 80% of citations (author, year, publication)
- Spot-check 50% of URLs

---

### Step 3: Decompose Complex Claims

Break compound claims into atomic facts:

**Example:**
> "The evaluation found participants were 58% less likely to fail"

Atomic facts:
1. There was an evaluation study
2. The metric was likelihood of failure
3. The reduction was 58%
4. This was comparing participants to non-participants

---

### Step 4: Verify Each Claim

For each claim, search for the original source using a **tiered approach**:

#### Tier 1: Local Source Library (if configured)

If project has a local search tool configured in CLAUDE.md, search there first:

```bash
# Example: Check if local service is available
curl -s http://localhost:[PORT]/health

# Search using project-configured tool
```

**Why local first?**
- Sources already vetted and in your collection
- Extracted text available for exact verification
- No rate limits or network latency

#### Tier 2: External Academic Search (if configured)

If not found locally and project has external search configured:

```bash
# Example via CLI tool
[tool] search "[author] [topic]" --year-from [year-1] --year-to [year+1]
```

**Benefits:**
- Access to large paper databases
- Can acquire new sources for future use

#### Tier 3: Web Search (always available)

```bash
# If URL provided
web_fetch([URL from footnote])

# If citation provided
web_search("site:scholar.google.com [author] [year] [topic]")
web_search("[exact title] filetype:pdf")

# For government statistics
web_search("site:bls.gov OR site:census.gov [topic]")

# For domain-specific sources (configure in CLAUDE.md)
web_search("site:[domain] [topic]")
```

Compare claim to source. Assign verdict:

| Verdict | Definition |
|---------|------------|
| **SUPPORTED** | Matches source exactly or within rounding |
| **PARTIALLY SUPPORTED** | Directionally correct but imprecise |
| **UNSUPPORTED** | Cannot find evidence |
| **CONTRADICTED** | Source says something different |

Rate confidence:
- **High**: Accessed original source
- **Medium**: Found corroborating secondary source
- **Low**: Only indirect evidence
- **Unable**: Could not locate source

---

### Step 5: Dynamic Parallel Execution (Default)

**Parallelization is the default behavior.** The command dynamically determines parallelization based on claim density.

**Part A: Quick Claim Count**

Before spawning agents, do a rapid scan to count verifiable claims per section:

```
For each section:
  1. Count lines with statistics (numbers, percentages, dollar amounts)
  2. Count citation patterns ([^footnote], author/year references)
  3. Count study references ("found that", "study", "research")
  4. Total = claim density estimate
```

**Threshold:** If a section has >15 estimated claims, subdivide it further before spawning agents.

**Part B: Detect Document Sections**

Scan the document for substantive divisions:

```bash
# Find all ## headings (major sections)
grep -n "^## " [file] | head -20
```

Identify discrete sections that can be checked independently.

**Part C: Adaptive Section Splitting**

Apply claim-based splitting:

```
For each section:
  If claim_count <= 15:
    → Assign to single agent
  If claim_count > 15 and claim_count <= 30:
    → Split into 2 agents (by line midpoint)
  If claim_count > 30:
    → Split into 3+ agents (~10-12 claims each)
```

**Example:**
```
Document analysis:
  - Part I: Background (lines 1-150) — 28 claims detected
    → Split into Part I-A (lines 1-75) and Part I-B (lines 76-150)
  - Part II: Methods (lines 151-300) — 12 claims detected
    → Single agent
  - Part III: Results (lines 301-400) — 8 claims detected
    → Single agent

Spawning 4 parallel agents...
```

**Part D: Spawn Agents**

Launch parallel agents using the Task tool:

```
Task tool parameters:
  - subagent_type: "general-purpose"
  - model: "sonnet"  # Faster, sufficient for fact-checking
  - prompt: [agent instructions]
```

Each subagent receives:
1. The specific line range to check
2. The target claim count (~10-15 claims max)
3. Instructions to output findings in standardized correction format

```
You are fact-checking [Document], lines [START]-[END] ([Section Name]).

This section contains approximately [N] verifiable claims. Verify each one.

Follow the methodology in this fact-check command.

Output your findings in the standardized corrections format with:
- Location as `file:line`
- Context (before) with 2-3 surrounding lines
- Suggested fix with same context
- Source citation
- Confidence level

Return your findings when complete.
```

**Part E: Synthesize Results**

After all agents complete:
1. Collect all findings from subagents
2. Deduplicate any overlapping claims
3. Sort by line number
4. Merge into single corrections report

**Important:** Launch all section agents in a **single message with multiple Task tool calls** to maximize parallelization.

---

### Step 6: Generate Report

Save the report to the configured report directory (default: `temp/`) with filename `fact-check-report-YYYY-MM-DD-HHMMSS.md`:

```markdown
## Corrections Report: [Document Name]

**Source:** /fact-check
**Generated:** [timestamp]
**Target:** [file path]

---

### Summary

| Verdict | Count |
|---------|-------|
| Supported | X |
| Partially Supported | X |
| Unsupported | X |
| Contradicted | X |
| Unable to Verify | X |

**Accuracy Rate:** X% of verifiable claims supported

| Category | Count |
|----------|-------|
| Errors (must fix) | X |
| Warnings (should fix) | X |
| Suggestions (consider) | X |

---

### Corrections

#### [1] [Short description]

- **Location:** `[file:line]`
- **Severity:** Error | Warning | Suggestion
- **Category:** Statistic | Citation | Study Finding | [Domain-specific]
- **Context (before):**
  ```
  [2-3 lines including the claim to correct]
  ```
- **Issue:** [What's wrong - the discrepancy]
- **Suggested fix:**
  ```
  [Corrected text with same context]
  ```
- **Source:** [Original source with citation]
- **Confidence:** High | Medium | Low

#### [2] [Short description]
...

---

### Verified Claims

| Line | Claim Summary | Source | Confidence |
|------|---------------|--------|------------|
| XX | [brief] | [citation] | High |

---

### Unable to Verify

| Line | Claim | Reason |
|------|-------|--------|
| XX | [claim] | [why couldn't verify] |

---

### Next Steps

Run `/apply [report-path]` to apply corrections automatically (if /apply command available).
```

---

### Step 7: Apply Corrections

**If `--report-only` was passed:**
- Present report only
- Suggest: "Run without --report-only to apply corrections"
- Stop here

**If corrections found:**
Follow the [correction-workflow](../guides/correction-workflow.md) guide.

**If no corrections needed:**
- Report: "All claims verified! No corrections needed."

---

### Step 8: Report Results

After user decision:

```markdown
## Fact-Check Complete

**Document:** [filename]
**Claims Verified:** X
**Corrections:** Y applied, Z skipped
**Status:** [Merged to main / Rejected / Kept for review]

### Accuracy Summary
- Supported: X
- Corrections applied: Y
- Unable to verify: Z

### Changes Made
- Line 89: 39% → 36% (source: Smith 2023)
- Line 156: Added citation

### Skipped
- Line 203: Unable to verify (source not found)
```

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

## Project Configuration

Projects can customize fact-checking by adding configuration to their CLAUDE.md. See [CONFIGURATION.md](../CONFIGURATION.md) for all available options, detailed descriptions, and examples.

---

## Flags

| Flag | Effect |
|------|--------|
| `--report-only` | Generate report without applying corrections |
| `--sequential` | Disable parallel execution |
| `--auto-approve` | Apply and merge without prompting |
| `--section "X"` | Only fact-check named section |

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

### 2.3.0 (2025-01-20)
- Extracted configuration examples to central CONFIGURATION.md
- Added reference to CONFIGURATION.md in Related section

### 2.2.0 (2025-01-20)
- Added Troubleshooting section

### 2.1.0 (2025-01-20)
- Standardized terminology: "Phase" → "Step"
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

- `/proof` - Formatting and style checks (different purpose)
- [CONFIGURATION.md](../CONFIGURATION.md) - All CLAUDE.md configuration options
- [correction-workflow](../guides/correction-workflow.md) - Shared apply logic
