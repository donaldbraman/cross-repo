# Lint & Hooks (Global)

**Read before:** Step 5 (Lint) - First commit attempt

## Critical Rule

❌ **NEVER** use `--no-verify` in autonomous mode

## When Hooks Fail

1. Read the error output
2. Fix the code that triggered it
3. Re-commit until hooks pass

## Common Fixes

### Missing Docstrings
Add to classes, `__init__`, and public methods:
```python
def __init__(self, agent):
    """Initialize with parent agent."""
    self.agent = agent
```

### Auto-Fix with Ruff
```bash
ruff check --fix .
ruff format .
git add -A && git commit -m "message"
```

### Line Too Long
Break into multiple lines or rephrase.

## Philosophy

Hooks are quality gates. Bypassing them = technical debt. Fix the code, don't skip the gate.

## Project-Specific

Check `docs/guides/lint-and-hooks.md` in current repo for project-specific hook details.
