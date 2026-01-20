# PR Guidelines

**Version:** 1.0.0
**Last Updated:** 2025-01-20
**Read before:** Step 6 (PR) - Creating pull request

## Title Format

```
<type>: <description>
```

**Types:** `feat`, `fix`, `refactor`, `docs`, `test`, `chore`

**Rules:**

- Lowercase after colon, no period
- Imperative mood: "Add feature" not "Added feature"
- Under 72 chars
- Issue number in body, not title

## Body Template

```markdown
## Summary
- What changed (2-4 bullets)
- Why it matters

## Testing
How verified

## Breaking Changes
[None or list changes]

Closes #XXX
```

**Example:**

```markdown
## Summary
- Implemented v3 API with hybrid search
- Searches both chunks and summaries
- Improves relevance for document-level queries

## Testing
```bash
curl -X POST http://localhost:8000/api/v3/search \
  -H "Content-Type: application/json" \
  -d '{"library_id": "5673253", "query": "test", "top_k": 5}'
# Returns in 2.3s
```

Tests: `tests/test_api_v3.py` - 127 passed

## Breaking Changes

None

Closes #442

```

## Create PR

```bash
# 1. Create labels (every time)
gh label create "enhancement" --color "a2eeef" --description "New feature" --force 2>/dev/null || true
gh label create "bug" --color "d73a4a" --description "Bug" --force 2>/dev/null || true

# 2. Create PR
gh pr create \
  --title "feat: Add hybrid search" \
  --label "enhancement" \
  --body "$(cat <<'EOF'
## Summary
- Added hybrid search

## Testing
All tests pass

Closes #442
EOF
)"
```

## Merge

```bash
# Autonomous mode (standard)
gh pr merge <number> --squash --auto --delete-branch

# Wait for review if: breaking changes, security, major refactor, CI/CD, migrations
gh label create "needs-review" --color "fbca04" --description "Needs review" --force 2>/dev/null || true
gh pr create --label "needs-review" --title "..." --body "..."  # No --auto
```

**Always squash merge:** Single commit on main, clean history.

## Troubleshooting

```bash
# PR creation fails
gh auth status
git branch --show-current

# Can't merge
gh pr checks <number>
gh pr view <number>
```

## Project-Specific

Check `docs/guides/pr-best-practices.md` for templates, reviewers, CI/CD, release process.

## Related Guides

- [Git Workflow](git-workflow.md) - Branch and commit conventions
- [GitHub Labels](github-labels.md) - Label creation and usage
- [Autonomous Cycle](autonomous-cycle.md) - Step 6: PR creation in full workflow

---

*This guide ensures consistent PR quality across all projects.*
