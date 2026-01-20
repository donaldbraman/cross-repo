# Testing Agent

**Version:** 2.1.0
**Created:** 2024-10-14
**Last Updated:** 2025-01-20
**Agent Type:** general-purpose

## Purpose

Autonomous test execution without polluting main agent context. Runs test suites, reports only failures and warnings using standardized format.

## Input Contract

**Required Parameters:**

- `test_suite`: Which test suite to run (unit|integration|api|e2e|performance|smoke|all)

**Optional Parameters:**

- `verbose`: Include passing tests in report (default: false)
- `timeout_override`: Custom timeout in seconds
- `test_path`: Custom path to tests (default: tests/)
- `package_manager`: Package manager to use (uv|npm|yarn|pnpm|cargo|go - auto-detected if not specified)

## Output Contract

**Success (all tests pass):**

```
All tests passed
Suite: [suite name]
Count: X/X tests
Duration: Xs
```

**Failures:**

```
TEST FAILURE
Suite: [suite name]
Test: test_name (file_path:line_number)
Error: <error type>
Message: <error message>
Context: <relevant params/state>
```

**Warnings:**

```
WARNING
Type: [deprecation|connection|timeout|performance]
Test: test_name
Message: <warning message>
Impact: [low|medium|high]
```

## Prompt Template

```
**Your Task:** Run {test_suite} tests and report results.

**Steps:**
1. Detect project type and test framework
2. Check service dependencies (if integration/API tests)
3. Run test suite with appropriate timeout
4. Parse output for failures and warnings
5. Report using standardized format above

**Report ONLY:**
- Test failures (all details)
- Warnings from project code (not dependencies)
- Service connection issues
- Performance degradation

**Do NOT report:**
- Passing tests (unless verbose=true)
- Expected warnings from dependencies
- Verbose test output

**Auto-Detection:**

Detect test framework from project files:
- `pyproject.toml` or `pytest.ini` -> pytest
- `package.json` with jest/vitest/mocha -> node test runner
- `Cargo.toml` -> cargo test
- `go.mod` -> go test

**Test Commands by Framework:**

Python (pytest):
```bash
uv run pytest {test_path} -v --tb=short
```

Node.js (jest/vitest):

```bash
npm test -- --testPathPattern={test_path}
```

Rust:

```bash
cargo test
```

Go:

```bash
go test ./...
```

**Suite Mappings:**

- `unit` -> tests/unit/ or tests that don't require services
- `integration` -> tests/integration/ or tests marked integration
- `api` -> tests/api/ or API-specific tests
- `e2e` -> tests/e2e/ or end-to-end tests
- `performance` -> tests/performance/ or benchmarks
- `smoke` -> quick sanity checks
- `all` -> all tests

**Timeouts:**

- Unit: 60s
- Integration: 180s
- E2E: 300s
- Performance: 600s
- Smoke: 60s

**Service Health Checks (if needed):**

Before running integration tests, check service dependencies:

```bash
# Generic health endpoint pattern
curl -s http://localhost:PORT/health || echo "Service at PORT not available"
```

**Your Final Report:**

Provide actionable summary:

- What failed (if anything)
- What warnings occurred (if any)
- Service status (if applicable)
- Recommended next steps

```

## Version History

### 2.1.0 (2025-01-20)
- Added Validation Checklist section with initial test case, success criteria, and common failure modes

### 2.0.0 (2025-01-19)
- Generalized for cross-repo sharing
- Added auto-detection of test frameworks
- Made package manager configurable
- Removed project-specific paths

### 1.1.0 (2024-10-14)
- Added v3 API test commands (unit and integration)
- Added combined v2+v3 API test command
- Updated test suite documentation for hybrid search testing

### 1.0.0 (2024-10-14)
- Initial implementation (cite-assist specific)
- Supports all test suites
- Standardized reporting format
- Service dependency checking

## Success Metrics

- **Context Saved:** >50% reduction in test output in main agent context
- **Accuracy:** 100% failure detection rate
- **Actionability:** All reports include next steps
- **Performance:** Report delivered within test timeout + 10s

## Usage Examples

### Run Unit Tests
```

Task tool with:

- description: "Run unit tests"
- subagent_type: "general-purpose"
- prompt: <contents with test_suite=unit>

```

### Run Integration Tests with Custom Timeout
```

Task tool with:

- description: "Run integration tests"
- subagent_type: "general-purpose"
- prompt: <contents with test_suite=integration, timeout_override=300>

```

### Run All Tests Verbosely
```

Task tool with:

- description: "Run all tests verbose"
- subagent_type: "general-purpose"
- prompt: <contents with test_suite=all, verbose=true>

```

## Validation Checklist

Use this checklist to verify the testing agent works correctly before relying on it.

### Initial Test Case

**Minimal verification test:**
```

Task tool with:

- description: "Verify testing agent"
- subagent_type: "general-purpose"
- prompt: "Run smoke tests and report results using test_suite=smoke"

```

**What to check:**
1. Agent detects project type correctly (Python/Node/Rust/Go)
2. Agent finds the test directory or reports if missing
3. Agent executes appropriate test command
4. Agent returns structured output (not raw test runner output)

### Expected Output Format

**Success case should include:**
- "All tests passed" or "TEST FAILURE" header
- Suite name identification
- Test count (X/X format)
- Duration in seconds

**Failure case should include:**
- Test name and file location
- Error type and message
- Relevant context for debugging

### Success Criteria

| Criterion | How to Verify |
|-----------|---------------|
| Test discovery | Agent finds tests in standard locations (tests/, src/tests/) |
| Framework detection | Agent uses correct runner (pytest, jest, cargo, go test) |
| Execution | Tests actually run (check for timing info in output) |
| Reporting | Output follows standardized format, not raw runner output |
| Failure capture | Intentionally failing test is reported with file:line |
| Timeout handling | Long-running test triggers timeout warning |

### Common Failure Modes and Fixes

| Failure Mode | Symptoms | Fix |
|--------------|----------|-----|
| No tests found | "0 tests collected" or empty report | Verify `test_path` parameter; check tests directory exists |
| Wrong framework | Command fails immediately | Check project files (pyproject.toml, package.json, etc.) exist |
| Missing dependencies | Import errors in test output | Run package install first (uv sync, npm install, etc.) |
| Service unavailable | Connection refused errors | Start required services before running integration tests |
| Timeout exceeded | Report never returns | Use `timeout_override` parameter or reduce test scope |
| Raw output leak | Seeing full pytest/jest output | Agent not parsing correctly; check prompt template |
| Permission denied | Test files inaccessible | Check file permissions on test directory |

### Quick Validation Script

Run these commands to validate agent readiness:

```bash
# 1. Verify test directory exists
ls -la tests/ 2>/dev/null || ls -la src/tests/ 2>/dev/null || echo "No standard test directory found"

# 2. Verify test framework config exists
[ -f pyproject.toml ] && echo "Python project detected"
[ -f package.json ] && echo "Node project detected"
[ -f Cargo.toml ] && echo "Rust project detected"
[ -f go.mod ] && echo "Go project detected"

# 3. Verify test command works manually
# Python: uv run pytest tests/ -v --collect-only
# Node: npm test -- --listTests
# Rust: cargo test --no-run
# Go: go test ./... -list '.*'
```

---

*Autonomous testing agent for consistent, actionable test reports.*
