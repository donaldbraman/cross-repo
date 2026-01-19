# Agent-to-Agent Messaging Protocol

**Version:** 2.0.0
**Last Updated:** 2025-11-10
**Applies to:** All AI agents across repositories

## Message Format

```markdown
**From**: `<agent-name>`
**To**: `<agent-name>`
**Priority**: Urgent | Normal | Info
**Topic**: Bug | Performance | Feature | Question | Info

---

[Message content]
```

**Example:**
```markdown
**From**: `pin-citer`
**To**: `cite-assist`
**Priority**: Urgent
**Topic**: Bug

---

Endpoint /api/v2/batch-search timeout 30s.
**Error:** HTTP 500 after 14s
**Impact:** 50% searches failing
**Request:** Check server logs
```

## Labels

**Required:**
- `agent-message`
- `to:<agent-name>`
- `from:<agent-name>`

**Create once:**
```bash
gh label create "agent-message" --color "0E8A16" --description "Agent message"
gh label create "to:cite-assist" --color "1D76DB" --description "To cite-assist"
gh label create "from:pin-citer" --color "FBCA04" --description "From pin-citer"
```

## Send Message

```bash
gh issue create \
  --repo owner/recipient-repo \
  --title "[ERROR] Service timeout" \
  --label "agent-message,to:cite-assist,from:pin-citer" \
  --body "**From**: \`pin-citer\`
**To**: \`cite-assist\`
**Priority**: Urgent
**Topic**: Bug

---

Endpoint timing out after 30s..."
```

## Receive Messages

**Auto-detection hook** (`.claude/hooks-agent-messaging.json`):

```json
{
  "$schema": "https://anthropic.com/claude-code/hooks-schema.json",
  "hooks": {
    "SessionStart": [
      {
        "name": "check-agent-messages",
        "command": "bash",
        "args": ["-c", "gh issue list --repo $(gh repo view --json nameWithOwner -q .nameWithOwner) --label 'agent-message,to:YOUR-AGENT-ID' --state open --json number,title,body,createdAt --jq '.[] | \"📬 Issue #\\(.number): \\(.title)\\nCreated: \\(.createdAt)\\n\\n\\(.body)\\n---\"' || true"],
        "timeout": 10000
      }
    ]
  }
}
```

Replace `YOUR-AGENT-ID` with agent name.

**Manual check:**
```bash
gh issue list --label "agent-message,to:cite-assist" --state open
```

## Priority

- **Urgent:** <4h response (blocking bugs, production issues)
- **Normal:** <24h response (performance, features, non-blocking bugs)
- **Info:** No response (status updates, FYI)

## Respond

```bash
# Comment
gh issue comment 381 --body "Resolved. Service restarted, timeout fixed."

# Close
gh issue close 381 --comment "Resolved. Timeout 30s→2.3s."
```

## Workflow

1. Add to todo
2. Investigate
3. Respond (comment or new issue)
4. Close original
5. Mark todo complete

## Troubleshooting

```bash
# Messages not appearing
gh label list --repo your-repo | grep agent-message
gh issue list --label "agent-message,to:your-agent" --repo target-repo
gh auth status

# Can't send
gh label list --repo target-repo | grep agent-message
gh repo view target-repo
```

## Related Guides

- [GitHub Labels](github-labels.md) - Label creation and usage

---

*For questions, create an issue with label `agent-message-system`*
