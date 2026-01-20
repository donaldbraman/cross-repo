# Lint & Hooks

**Version:** 1.0.0
**Last Updated:** 2025-01-20
**Read before:** Step 5 (Lint) - First commit attempt

## Critical Rule

❌ **NEVER** use `--no-verify`
✅ **ALWAYS** fix the code

## Common Hooks

**Python:** ruff, mypy
**JS/TS:** eslint, prettier, tsc
**Other:** pytest, detect-secrets, trailing-whitespace

## When Hooks Fail

1. Read error output
2. Fix the code
3. Re-commit

```bash
git add <fixed-files>
git commit -m "message"
```

## Common Fixes

**D103 (Missing docstring):**

```python
def search_chunks(query: str, limit: int):
    """Search chunks and return top results."""
    results = db.query(query)
    return results[:limit]
```

**E501 (Line too long):**

```python
result = calculate_score(
    query_embedding,
    chunk_embedding,
    boost_factor,
    normalization_constant
)
```

**I001 (Import order):**

```bash
ruff check --fix .
```

**Type errors:**

```python
def get_chunks(ids: list[int]) -> list[Chunk]:
    return db.fetch(ids)
```

## Auto-Fix Pattern

If hook auto-fixes files:

```bash
# Commit triggers hook, modifies files
git commit -m "feat: Add feature"

# Amend if: (1) you're author, (2) not pushed yet
git add -A
git commit --amend --no-edit

# Otherwise: new commit
git add -A
git commit -m "style: Apply auto-fixes"
```

## Troubleshooting

```bash
# Hooks not running
ls .git/hooks/pre-commit
pre-commit install
pre-commit run --all-files

# Error help
# 1. Search error code (e.g., "ruff D103")
# 2. Check docs/guides/lint-and-hooks.md
```

## Project-Specific

Check `docs/guides/lint-and-hooks.md` for project-specific hooks, rules, workarounds.

## Related Guides

- [Git Workflow](git-workflow.md) - Commit conventions and workflow
- [Autonomous Cycle](autonomous-cycle.md) - Step 5: Lint in full workflow

---

*This guide applies to all projects. Never bypass quality gates.*
