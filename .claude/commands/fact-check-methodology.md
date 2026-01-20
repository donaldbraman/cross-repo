# Fact-Check Methodology

**Version:** 1.0.0
**Last Updated:** 2025-01-20

Detailed step-by-step methodology for fact-checking documents. This is a companion file to the main [fact-check.md](fact-check.md) command.

---

## Step 0: Configuration

1. **Check CLAUDE.md for project-specific settings:**
   - Verification tools (local search APIs, external databases)
   - Domain-specific claim categories
   - Source file locations
   - Report output directory

2. **Check for linked style guides** that may define citation formats or terminology standards.

See [CONFIGURATION.md](../CONFIGURATION.md) for all available options and examples.

If no configuration found, use defaults (web search only, standard categories).

---

## Step 1: Extract Claims

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

## Step 2: Prioritize

Rank claims by importance:

```
Priority = (Central to argument) x (Specificity) x (Verifiability)
```

For a typical document:
- Verify 100% of statistics and effect sizes
- Verify 100% of study characterizations supporting key arguments
- Verify 80% of citations (author, year, publication)
- Spot-check 50% of URLs

---

## Step 3: Decompose Complex Claims

Break compound claims into atomic facts:

**Example:**
> "The evaluation found participants were 58% less likely to fail"

Atomic facts:
1. There was an evaluation study
2. The metric was likelihood of failure
3. The reduction was 58%
4. This was comparing participants to non-participants

---

## Step 4: Verify Each Claim

For each claim, search for the original source using a **tiered approach**:

### Tier 1: Local Source Library (if configured)

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

### Tier 2: External Academic Search (if configured)

If not found locally and project has external search configured:

```bash
# Example via CLI tool
[tool] search "[author] [topic]" --year-from [year-1] --year-to [year+1]
```

**Benefits:**
- Access to large paper databases
- Can acquire new sources for future use

### Tier 3: Web Search (always available)

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

## Step 5: Dynamic Parallel Execution (Default)

**Parallelization is the default behavior.** The command dynamically determines parallelization based on claim density.

### Part A: Quick Claim Count

Before spawning agents, do a rapid scan to count verifiable claims per section:

```
For each section:
  1. Count lines with statistics (numbers, percentages, dollar amounts)
  2. Count citation patterns ([^footnote], author/year references)
  3. Count study references ("found that", "study", "research")
  4. Total = claim density estimate
```

**Threshold:** If a section has >15 estimated claims, subdivide it further before spawning agents.

### Part B: Detect Document Sections

Scan the document for substantive divisions:

```bash
# Find all ## headings (major sections)
grep -n "^## " [file] | head -20
```

Identify discrete sections that can be checked independently.

### Part C: Adaptive Section Splitting

Apply claim-based splitting:

```
For each section:
  If claim_count <= 15:
    -> Assign to single agent
  If claim_count > 15 and claim_count <= 30:
    -> Split into 2 agents (by line midpoint)
  If claim_count > 30:
    -> Split into 3+ agents (~10-12 claims each)
```

**Example:**
```
Document analysis:
  - Part I: Background (lines 1-150) -- 28 claims detected
    -> Split into Part I-A (lines 1-75) and Part I-B (lines 76-150)
  - Part II: Methods (lines 151-300) -- 12 claims detected
    -> Single agent
  - Part III: Results (lines 301-400) -- 8 claims detected
    -> Single agent

Spawning 4 parallel agents...
```

### Part D: Spawn Agents

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

### Part E: Synthesize Results

After all agents complete:
1. Collect all findings from subagents
2. Deduplicate any overlapping claims
3. Sort by line number
4. Merge into single corrections report

**Important:** Launch all section agents in a **single message with multiple Task tool calls** to maximize parallelization.

---

## Step 6: Generate Report

Save the report to the configured report directory (default: `temp/`) with filename `fact-check-report-YYYY-MM-DD-HHMMSS.md`.

Format corrections following the [Correction Report Schema](../templates/correction-report-schema.md). The fact-check report extends the base schema with:

1. **Header** with source (`/fact-check`), timestamp, and target file path
2. **Summary tables:**
   - Verdict counts (Supported, Partially Supported, Unsupported, Contradicted, Unable to Verify)
   - Accuracy rate percentage
   - Severity counts (Errors, Warnings, Suggestions)
3. **Corrections section** with each item containing:
   - Standard fields: numbered description, Location, Severity, Context (before), Suggested fix, Rationale
   - Fact-check specific fields: **Category** (Statistic, Citation, Study Finding, etc.), **Issue** (the discrepancy), **Source** (original citation), **Confidence** (High, Medium, Low)
4. **Verified Claims table** - claims that passed verification
5. **Unable to Verify table** - claims that could not be checked

---

## Step 7: Apply Corrections

**If `--report-only` was passed:**
- Present report only
- Suggest: "Run without --report-only to apply corrections"
- Stop here

**If corrections found:**
Follow the [correction-workflow](../guides/correction-workflow.md) guide.

**If no corrections needed:**
- Report: "All claims verified! No corrections needed."

---

## Step 8: Report Results

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
- Line 89: 39% -> 36% (source: Smith 2023)
- Line 156: Added citation

### Skipped
- Line 203: Unable to verify (source not found)
```

---

## Related

- [fact-check](fact-check.md) - Main command file
- [fact-check-reference](fact-check-reference.md) - Claim types and red flags
- [Correction Report Schema](../templates/correction-report-schema.md) - Standard format for corrections
- [correction-workflow](../guides/correction-workflow.md) - Shared apply logic
