# GitHub Labels

**Version:** 1.0.0
**Last Updated:** 2025-01-20
**Applies to:** All repositories

## Workflow: Create Before Use

**Always create labels before using them** (idempotent):

```bash
# 1. CREATE (--force updates existing, || true never fails)
gh label create "bug" --color "d73a4a" --description "Something isn't working" --force 2>/dev/null || true
gh label create "enhancement" --color "a2eeef" --description "New feature" --force 2>/dev/null || true

# 2. USE
gh issue create --label "bug" --title "fix: Login error" --body "..."
```

## Core Labels

```bash
gh label create "bug" --color "d73a4a" --description "Something isn't working" --force 2>/dev/null || true
gh label create "enhancement" --color "a2eeef" --description "New feature" --force 2>/dev/null || true
gh label create "documentation" --color "0075ca" --description "Documentation" --force 2>/dev/null || true
gh label create "refactor" --color "fbca04" --description "Code improvement" --force 2>/dev/null || true
```

**Use ONE type label per issue/PR:**
- `bug` - Fixes broken functionality
- `enhancement` - New feature
- `documentation` - Docs only
- `refactor` - Code improvement, no behavior change

## Complete Example

```bash
# Create labels (run every time)
gh label create "bug" --color "d73a4a" --description "Bug" --force 2>/dev/null || true
gh label create "priority:high" --color "d93f0b" --description "High priority" --force 2>/dev/null || true

# Create issue
gh issue create --label "bug,priority:high" --title "fix: Login bug" --body "..."
```

## Conventions

- Lowercase with hyphens: `needs-review`
- Namespaces for categories: `priority:high`, `status:blocked`

## Search

```bash
gh search issues "label:bug created:>2024-01" --repo owner/repo
gh issue list --label "enhancement" --state open
```

## Project-Specific

Check `docs/guides/` for additional labels (agent-message, status, priority).

## Troubleshooting

```bash
# List labels
gh label list --repo owner/repo

# Delete label
gh label delete "wrong-label" --repo owner/repo
```

## Related Guides

- [Agent Messaging](agent-messaging.md) - Agent-specific message labels
- [PR Guidelines](pr-guidelines.md) - PR label conventions

---

*Labels enable powerful search and organization - create them before every use*
