# Testing Agent

**Version:** 2.0.0
**Created:** 2024-10-14
**Last Updated:** 2025-01-19
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

---

*Autonomous testing agent for consistent, actionable test reports.*
