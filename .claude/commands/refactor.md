# Refactor Command

**Version:** 1.0.0
**Last Updated:** 2025-01-20

Analyze a codebase for refactoring opportunities, rank them by priority, and optionally execute them via `/dev-cycle` subagents. Use `$ARGUMENTS` to specify options.

## Usage

| Command | Action |
|---------|--------|
| `/refactor` | Analyze, rank, and execute refactorings (with user approval each) |
| `/refactor --report` | Analyze and produce ranked report only (no execution) |
| `/refactor --path <dir>` | Limit analysis to specific path |
| `/refactor --auto-approve` | Execute all refactorings without per-item confirmation |
| `/refactor --max N` | Limit execution to top N refactorings |

Options can be combined: `/refactor --path src/utils --max 3 --auto-approve`

## Instructions

You are executing a codebase refactoring analysis and execution workflow.

---

### Step 1: Parse Arguments

Parse `$ARGUMENTS` for the following flags:

| Flag | Default | Description |
|------|---------|-------------|
| `--report` | false | Only produce report, no execution |
| `--path <dir>` | `.` (full codebase) | Limit analysis scope |
| `--auto-approve` | false | Skip confirmation prompts |
| `--max N` | unlimited | Maximum refactorings to execute |

**Output:** Confirm parsed options before proceeding.

---

### Step 2: Codebase Analysis

Use the Explore agent (Task tool with subagent_type=Explore) to scan the codebase (or `--path` scope) for refactoring opportunities.

**Categories to identify:**

| Category | Description |
|----------|-------------|
| **Code Duplication** | Repeated code blocks that could be extracted |
| **Long/Complex Functions** | Functions with high cyclomatic complexity (>10) or length (>50 lines) |
| **Dead Code** | Unused exports, unreachable code, commented-out blocks |
| **Inconsistent Patterns** | Mixed conventions, naming inconsistencies |
| **Missing Abstractions** | Repeated patterns that need a shared utility |
| **Type Safety Gaps** | Missing types, `any` usage, unsafe casts |
| **Test Coverage Gaps** | Untested critical paths, missing edge cases |

**Analysis approach:**
1. Read directory structure to understand project layout
2. Identify entry points and core modules
3. Examine each module for the categories above
4. Note specific locations and code snippets

---

### Step 3: Prioritization

Rank each opportunity using three criteria:

| Criterion | High | Medium | Low |
|-----------|------|--------|-----|
| **Impact** | Core functionality, multiple consumers | Single feature, few consumers | Peripheral code, isolated |
| **Risk** | Breaking changes, complex dependencies | Some testing needed | Isolated, easy to test |
| **Effort** | Multi-file, architectural | Single file, moderate | Few lines, straightforward |

**Priority formula:**
- **Top Priority:** High impact + Low risk + Low effort
- **High Priority:** High impact + Medium risk OR Medium impact + Low risk
- **Medium Priority:** Medium impact + Medium risk/effort
- **Low Priority:** Low impact OR High risk OR High effort

Sort all opportunities by priority (top to bottom).

---

### Step 4: Generate Report

For each refactoring opportunity, produce:

```
## [Priority Rank] [Category]: [Short Title]

**Location:** `file:line`
**Priority:** [1-N] | Impact: [H/M/L] | Risk: [H/M/L] | Effort: [H/M/L]

**Issue:**
[What's wrong with the current code]

**Suggestion:**
[How to fix it - be specific]

**Rationale:**
[Why this matters for maintainability/reliability]

---
```

**Output:** Display the full ranked report.

---

### Step 5: Execution (unless --report)

1. **If `--report` flag is set:**
   - Display report and stop
   - Inform user they can re-run without `--report` to execute

2. **Otherwise, execute refactorings:**
   - Apply `--max N` limit if set (use top N items only)
   - For each refactoring in priority order:

   **Without `--auto-approve`:**
   ```
   Ready to execute refactoring [N]: [Title]
   Location: [file:line]

   Proceed? [Y/n/skip all/stop]
   ```
   Use AskUserQuestion tool with options:
   - Yes, execute this refactoring
   - No, skip this one
   - Skip all remaining
   - Stop execution

   **With `--auto-approve`:**
   - Proceed directly without confirmation

3. **Execute each approved refactoring:**
   - Spawn a Task tool subagent (general-purpose type)
   - The subagent should execute `/dev-cycle` with a specific task description:
     ```
     Task: [Category] refactoring - [Short description]

     Location: [file:line]

     Required changes:
     [Specific changes from the Suggestion section]

     Acceptance criteria:
     - Code compiles/lints successfully
     - Tests pass
     - No functionality changes (unless fixing bugs)
     ```
   - Wait for subagent completion
   - Report result (success/failure) before moving to next item

4. **Summary:**
   After all executions (or early stop), provide:
   - Total refactorings identified
   - Number executed
   - Number succeeded
   - Number failed (with reasons)
   - Number skipped

---

## Critical Rules

- **NEVER** modify code directly - always use `/dev-cycle` subagents
- **NEVER** execute without user approval unless `--auto-approve` is set
- **ALWAYS** report failures immediately
- **ALWAYS** preserve existing functionality (refactoring only, not feature changes)
- **ALWAYS** ensure tests pass before marking success

---

## Troubleshooting

### Analysis finds too many issues
Use `--path` to narrow scope or `--max` to limit execution.

### Subagent fails to complete
- Check the specific error in subagent output
- May need manual intervention for complex refactorings
- Consider breaking into smaller changes

### False positives in analysis
Skip those items during execution. Future versions may add exclusion patterns.

---

## Completion Checklist

Before reporting completion, verify:
- [ ] Arguments parsed correctly
- [ ] Analysis covered intended scope
- [ ] Report is clear and actionable
- [ ] (If executing) Each refactoring used `/dev-cycle`
- [ ] (If executing) Summary provided

---

## Related

- [dev-cycle](dev-cycle.md) - Autonomous development cycle (used for execution)
- [autonomous-cycle](../guides/autonomous-cycle.md) - Detailed worktree workflow
- [lint-and-hooks](../guides/lint-and-hooks.md) - Pre-commit hook handling

---

## Version History

### 1.0.0 (2025-01-20)
- Initial release
- Analysis for 7 refactoring categories
- Priority-based ranking
- Optional execution via `/dev-cycle` subagents
- Support for `--report`, `--path`, `--max`, `--auto-approve` flags
