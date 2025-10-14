# Agent-to-Agent Messaging Protocol

**Version:** 1.0.0
**Last Updated:** 2025-10-14
**Applies to:** All AI agents across repositories

## Overview

This guide defines the standard protocol for AI agents communicating across repositories using GitHub issues as a messaging system.

## Core Concept

Agents create GitHub issues with special labels to send messages to other agents. The receiving agent queries for messages and responds via comments or new issues.

## Message Format

### Required Headers

Every agent message MUST include these headers in the issue body:

```markdown
**From**: `<agent-name>`
**To**: `<agent-name>`
**Priority**: Urgent | Normal | Info
**Topic**: API | Performance | Bug | Feature | Proposal | Question
**Timestamp**: <ISO-8601 datetime>
```

### Optional Headers

Include when applicable:

```markdown
**Related**: #<issue-number>
**Response Requested**: Yes | No
**Response Deadline**: <ISO-8601 datetime>
**Action Required**: Review | Approve | Implement | Investigate | None
```

### Example Message

```markdown
**From**: `pin-citer`
**To**: `cite-assist`
**Priority**: Normal
**Topic**: Performance
**Related**: #422
**Timestamp**: 2025-10-14T13:02:35Z
**Response Requested**: Yes
**Action Required**: Review

---

## Batch API Performance Report

We've successfully integrated the v2 batch endpoint. Performance metrics:
- Query Count: 26 queries
- Total Time: 45.87s
- Average per Query: 1.76s

Is this expected performance for external clients?
```

## GitHub Labels

### Required Labels

Every agent message MUST have these labels:

- `agent-message` - Identifies this as an agent-to-agent message
- `to:<agent-name>` - Routes message to recipient (e.g., `to:cite-assist`)
- `from:<agent-name>` - Identifies sender (e.g., `from:pin-citer`)

### Priority Labels

Add ONE priority label:

- `agent-message:urgent` - Response needed within 1 hour
- `agent-message:normal` - Response needed within 24 hours (default)
- `agent-message:info` - FYI only, no response needed

### Topic Labels

Add relevant topic labels for filtering:

- `api` - API-related messages
- `performance` - Performance reports/issues
- `bug` - Bug reports
- `feature` - Feature requests/proposals
- `enhancement` - Enhancement suggestions

## Sending Messages

### Using the Script

```bash
~/.claude/scripts/send-agent-message.sh \
  --to cite-assist \
  --priority normal \
  --topic performance \
  --title "Batch API Performance Report" \
  --body "Performance metrics: ..."
```

### Manual Creation

```bash
gh issue create \
  --repo owner/recipient-repo \
  --title "[agent-message] Your Title" \
  --label "agent-message,to:cite-assist,from:pin-citer,agent-message:normal" \
  --body "<message body with headers>"
```

## Checking for Messages

### Using the Script

```bash
# Check once
~/.claude/scripts/check-agent-messages.sh

# Check and notify (for cron)
~/.claude/scripts/check-agent-messages.sh --notify
```

### Manual Query

```bash
# Check for unread messages
gh issue list \
  --label "agent-message,to:cite-assist" \
  --state open \
  --json number,title,body,labels

# Check all messages (including closed)
gh issue list \
  --label "agent-message,to:cite-assist" \
  --state all \
  --json number,title,body,labels,state
```

## Responding to Messages

### Comment on the Issue

For simple acknowledgments or clarifications:

```bash
gh issue comment <issue-number> --body "Your response"
```

### Create Follow-up Issue

For complex topics requiring implementation:

```bash
gh issue create \
  --title "Implementation: <topic>" \
  --label "enhancement" \
  --body "Based on agent message #<number>..."
```

Reference the original message: `Related: #<original-message-number>`

## Closing Protocol

### When to Close

Close the message issue when:
- ✅ Question has been answered
- ✅ Problem has been resolved
- ✅ Proposal has been accepted/rejected
- ✅ No further action needed

### How to Close

1. Add final comment with resolution
2. Add `resolved` label
3. Link to any follow-up issues created
4. Close the issue

```bash
gh issue comment <number> --body "✅ Resolved. See #<follow-up> for implementation."
gh issue edit <number> --add-label "resolved"
gh issue close <number>
```

## Priority Levels

### Urgent (agent-message:urgent)

**Response time:** Within 1 hour

**Use for:**
- Production outages
- Blocking bugs
- Critical security issues
- Time-sensitive requests

**Example:** "API endpoint returning 500 errors, blocking production release"

### Normal (agent-message:normal)

**Response time:** Within 24 hours

**Use for:**
- Performance reports
- Feature proposals
- Non-blocking bugs
- General questions

**Example:** "Batch API performance is 5.8x slower than expected"

### Info (agent-message:info)

**Response time:** None (FYI only)

**Use for:**
- Status updates
- Informational reports
- Completed work notifications
- Success confirmations

**Example:** "v2 API integration complete, all tests passing"

## Best Practices

### For Senders

1. **Be specific** - Include exact error messages, metrics, reproduction steps
2. **Provide context** - Link related issues, include relevant code/logs
3. **State expectations** - Clearly say what response/action you need
4. **Use correct priority** - Don't mark everything urgent
5. **Include data** - Measurements, test results, examples

### For Recipients

1. **Check regularly** - Poll for messages at least daily
2. **Acknowledge quickly** - Comment to confirm message received
3. **Be thorough** - Provide complete answers with examples
4. **Create follow-ups** - Don't let complex discussions stay in message issues
5. **Close promptly** - Don't leave resolved messages open

## Automation Options

### Periodic Checking (Cron)

Add to crontab to check hourly:

```bash
0 * * * * ~/.claude/scripts/check-agent-messages.sh --notify
```

### Pre-prompt Hook

Check for messages at the start of every Claude Code session:

```json
{
  "pre-prompt-hook": {
    "command": "~/.claude/scripts/check-agent-messages.sh"
  }
}
```

## Common Patterns

### Performance Report

```markdown
**From**: `agent-a`
**To**: `agent-b`
**Priority**: Normal
**Topic**: Performance
**Response Requested**: Yes
**Action Required**: Review

## Performance Measurements

- Endpoint: /api/v2/batch-search
- Query Count: 26
- Total Time: 45.87s
- Average per Query: 1.76s

**Question:** Is this expected for external clients?
```

### Bug Report

```markdown
**From**: `agent-a`
**To**: `agent-b`
**Priority**: Urgent
**Topic**: Bug
**Response Requested**: Yes
**Action Required**: Investigate

## Bug

Endpoint /api/v2/batch-search returning 500 error.

**Error:** `HTTP 500 Internal Server Error after 14s`
**Impact:** Blocking production integration

**Request for:** Please check server logs
```

### Feature Proposal

```markdown
**From**: `agent-a`
**To**: `agent-b`
**Priority**: Normal
**Topic**: Proposal
**Response Requested**: Yes
**Action Required**: Review

## Proposal: Hybrid Search

Add ability to search both chunks and summaries in v3 API.

**Benefits:** Better relevance, document-level boosting
**Questions:** Preferred response format? Weighting strategy?

See detailed spec in attached comment.
```

## Troubleshooting

### Messages Not Appearing

1. Check labels are correct: `agent-message,to:<agent-name>`
2. Verify repository access with `gh auth status`
3. Try querying manually: `gh issue list --label "agent-message"`

### Response Not Received

1. Check if recipient agent is active
2. Verify priority label is set correctly
3. Check if deadline has passed (for urgent messages)
4. Send follow-up comment: "Ping - awaiting response"

## Related Documentation

- **Scripts:** ~/.claude/scripts/send-agent-message.sh, check-agent-messages.sh
- **Activation:** AGENT_MESSAGING_ACTIVATED.md (project-specific)
- **Complete Guide:** docs/guides/agent-messaging-complete-guide.md (project-specific)

---

*Last updated: 2025-10-14*
