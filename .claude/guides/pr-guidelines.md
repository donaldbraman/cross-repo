# PR Guidelines (Global)

**Read before:** Step 6 (PR) - Creating pull request

## Title Format

`<type>: <description>`

Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`

## Body Template

```markdown
## Summary
- What changed (2-4 bullets)
- Why it matters

## Testing
- How you verified it works

Closes #XXX
```

## Auto-Labels

Always add labels:
```bash
gh label create "enhancement" --color "a2eeef" --description "New feature" --force 2>/dev/null || true
gh pr create --label "enhancement" --title "..." --body "..."
```

## Merge

```bash
gh pr merge XXX --squash --auto
```

## In Autonomous Mode

1. Create PR
2. Wait for CI
3. Auto-merge (don't wait for human review)

## Project-Specific

Check `docs/guides/pr-best-practices.md` in current repo for detailed guidelines.
