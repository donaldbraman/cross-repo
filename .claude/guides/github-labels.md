# GitHub Labels (Global)

**Applies to:** All repositories

## Auto-Generate Labels

Always create labels when making issues/PRs:

```bash
gh label create "bug" --color "d73a4a" --description "Something isn't working" --force 2>/dev/null || true
gh label create "enhancement" --color "a2eeef" --description "New feature or request" --force 2>/dev/null || true
gh label create "documentation" --color "0075ca" --description "Improvements or additions to documentation" --force 2>/dev/null || true
gh label create "refactor" --color "fbca04" --description "Code improvement without functionality change" --force 2>/dev/null || true
```

## Label Selection

- `bug` - Fixes something broken
- `enhancement` - New feature
- `documentation` - Docs only
- `refactor` - No behavior change

## Why

Labels enable historical search: `gh search issues "label:bug created:2024"`

## Pattern

```bash
# Create if missing (idempotent)
gh label create "bug" --color "d73a4a" --description "..." --force 2>/dev/null || true

# Apply to issue/PR
gh issue create --label "bug" --title "..." --body "..."
```
