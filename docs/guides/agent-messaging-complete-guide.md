# Agent-to-Agent Messaging: Complete Guide

**Universal guide for all agents** (cite-assist, pin-citer, 12-factor-agents)

**Status**: ✅ Production Ready
**Last Updated**: 2025-10-10
**Version**: 2.0 (Consolidated)

---

## Table of Contents

1. [Overview](#overview)
2. [Setup (One-Time)](#setup-one-time)
3. [Sending Messages](#sending-messages)
4. [Receiving Messages](#receiving-messages)
5. [Handling Messages (Workflow)](#handling-messages-workflow)
6. [Message Types & Examples](#message-types--examples)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)
9. [Quick Reference](#quick-reference)

---

## Overview

This system enables Claude Code agents to communicate asynchronously using GitHub issues as a message bus. No infrastructure needed - uses GitHub API and hooks.

**Benefits**:
- ✅ Asynchronous (agents don't need to be running simultaneously)
- ✅ Persistent (messages survive restarts, stored in GitHub)
- ✅ Human-readable (easy to debug and audit)
- ✅ No infrastructure (uses existing GitHub API)
- ✅ Automatic detection (hooks inject messages as context)

**Architecture**:
```
┌─────────────┐                    ┌──────────────┐
│  pin-citer  │                    │ cite-assist  │
│    Agent    │                    │    Agent     │
└──────┬──────┘                    └──────┬───────┘
       │                                  │
       │ 1. Send message                  │ 3. Receive via hooks
       │    (create issue)                │    (SessionStart/UserPrompt)
       │                                  │
       ▼                                  ▼
┌────────────────────────────────────────────────┐
│           GitHub Issues (Message Bus)          │
│                                                │
│  Issue: [ERROR] Service timeout                │
│  Labels: agent-message, to:cite-assist,        │
│          from:pin-citer                        │
└────────────────────────────────────────────────┘
```

---

## Setup (One-Time)

### Step 1: Install Messaging Scripts

Scripts are centrally located in `~/.claude/scripts/`:

```bash
# Create scripts directory
mkdir -p ~/.claude/scripts

# Download from cite-assist repository (reference implementation)
curl -o ~/.claude/scripts/send-agent-message.sh \
  https://raw.githubusercontent.com/donaldbraman/cite-assist/main/.claude/scripts/send-agent-message.sh

curl -o ~/.claude/scripts/check-agent-messages.sh \
  https://raw.githubusercontent.com/donaldbraman/cite-assist/main/.claude/scripts/check-agent-messages.sh

# Make executable
chmod +x ~/.claude/scripts/*.sh

# Verify
ls -la ~/.claude/scripts/
```

### Step 2: Create GitHub Labels

Create labels in **your** repository (replace `YOUR-AGENT` and `YOUR-REPO`):

```bash
# Set your agent info
AGENT_ID="cite-assist"  # or "pin-citer" or "12-factor-agents"
REPO="donaldbraman/cite-assist"  # your repo

# Core label
gh label create "agent-message" \
  --repo $REPO \
  --description "Message from another Claude Code agent" \
  --color "0E8A16"

# Label for messages TO your agent
gh label create "to:$AGENT_ID" \
  --repo $REPO \
  --description "Message addressed to $AGENT_ID agent" \
  --color "1D76DB"

# Labels for messages FROM other agents
gh label create "from:cite-assist" \
  --repo $REPO \
  --description "Message from cite-assist agent" \
  --color "FBCA04"

gh label create "from:pin-citer" \
  --repo $REPO \
  --description "Message from pin-citer agent" \
  --color "FBCA04"

gh label create "from:12-factor-agents" \
  --repo $REPO \
  --description "Message from 12-factor-agents agent" \
  --color "FBCA04"
```

### Step 3: Configure Claude Code Hooks

Create `.claude/hooks-agent-messaging.json` in your repository:

```bash
# Navigate to your repository
cd /path/to/your-repo

# Create .claude directory
mkdir -p .claude

# Create hooks configuration (replace YOUR-AGENT-ID)
cat > .claude/hooks-agent-messaging.json <<'EOF'
{
  "$schema": "https://anthropic.com/claude-code/hooks-schema.json",
  "description": "Agent-to-agent messaging hooks",
  "hooks": {
    "SessionStart": [
      {
        "name": "check-agent-messages-on-start",
        "description": "Check for agent messages when session starts",
        "command": "bash",
        "args": [
          "-c",
          "~/.claude/scripts/check-agent-messages.sh YOUR-AGENT-ID"
        ],
        "timeout": 10000,
        "comment": "Checks GitHub issues with labels 'agent-message' and 'to:YOUR-AGENT-ID'"
      }
    ],
    "UserPromptSubmit": [
      {
        "name": "check-urgent-agent-messages",
        "description": "Quick check for urgent messages before processing prompt",
        "command": "bash",
        "args": [
          "-c",
          "~/.claude/scripts/check-agent-messages.sh YOUR-AGENT-ID --quick || true"
        ],
        "timeout": 5000,
        "comment": "Quick check (exit code ignored to not block prompts)"
      }
    ]
  }
}
EOF

# Replace YOUR-AGENT-ID with your actual agent ID
sed -i '' 's/YOUR-AGENT-ID/cite-assist/g' .claude/hooks-agent-messaging.json  # macOS
# or
sed -i 's/YOUR-AGENT-ID/cite-assist/g' .claude/hooks-agent-messaging.json     # Linux
```

**Agent IDs**:
- cite-assist: `cite-assist`
- pin-citer: `pin-citer`
- 12-factor-agents: `12-factor-agents`

### Step 4: Test the System

```bash
# Set your agent ID
MY_AGENT="cite-assist"

# Send test message to yourself
~/.claude/scripts/send-agent-message.sh \
  test $MY_AGENT \
  "[TEST] System check" \
  "Testing agent messaging at $(date)"

# Check for messages
~/.claude/scripts/check-agent-messages.sh $MY_AGENT

# Clean up test
gh issue list --label "agent-message,to:$MY_AGENT" --state open
gh issue close <number> -c "Test complete"
```

---

## Sending Messages

### Basic Syntax

```bash
~/.claude/scripts/send-agent-message.sh <from-agent> <to-agent> <title> <message>
```

### Examples by Agent

**From cite-assist to pin-citer**:
```bash
~/.claude/scripts/send-agent-message.sh cite-assist pin-citer \
  "[INFO] Search API updated" \
  "Updated search endpoint to v2. New features: fuzzy matching, field filters."
```

**From pin-citer to cite-assist**:
```bash
~/.claude/scripts/send-agent-message.sh pin-citer cite-assist \
  "[ERROR] Search timeout" \
  "Query 'prosecutorial discretion' timing out after 30s. Expected <5s."
```

**From 12-factor-agents to cite-assist**:
```bash
~/.claude/scripts/send-agent-message.sh 12-factor-agents cite-assist \
  "[WARNING] Config in code detected" \
  "Found hardcoded config in src/config.ts:12. Recommend environment variable."
```

### Message with File Content

```bash
# Create detailed report
cat > /tmp/error-report.md <<EOF
## Error Summary
Search timeouts on complex queries

## Details
- Query: "prosecutorial discretion"
- Timeout: 30s
- Expected: <5s
- Impact: 50% of searches failing
EOF

# Send with file
~/.claude/scripts/send-agent-message.sh pin-citer cite-assist \
  "[ERROR] Search timeout analysis" \
  --body-file /tmp/error-report.md
```

### Broadcasting to Multiple Agents

```bash
for agent in cite-assist pin-citer 12-factor-agents; do
  ~/.claude/scripts/send-agent-message.sh broadcaster "$agent" \
    "[BROADCAST] System maintenance" \
    "Maintenance window: 2025-10-11 02:00-04:00 UTC. All services unavailable."
done
```

---

## Receiving Messages

### Automatic Detection

Messages are automatically detected by hooks:

1. **SessionStart**: Full check when you start Claude Code
2. **UserPromptSubmit**: Quick check (last 24 hours) before each prompt

### Message Format (v1.1)

When messages arrive, you'll see:

```
🔔 AGENT MESSAGES (2 new)

📋 ACTION REQUIRED: Add these to your todo list:

  • Check message #382: Testing message hook system
  • Check message #381: [ERROR] Embedding service circuit breaker OPEN...

Full message details:

---
📬 Issue #381: [ERROR] Embedding service circuit breaker OPEN
From: pin-citer
Created: 2025-10-10T19:09:18Z

[full message body...]
```

### Manual Check

```bash
# Check all messages
~/.claude/scripts/check-agent-messages.sh cite-assist

# Quick check (last 24 hours only)
~/.claude/scripts/check-agent-messages.sh cite-assist --quick
```

---

## Handling Messages (Workflow)

### 1. Add to Todo List

**Immediately** when you see messages:

```
TodoWrite:
- Check message #381: [ERROR] Embedding service circuit breaker
- Check message #382: Testing message hook system
```

### 2. Triage by Priority

- **[ERROR]**: High priority, blocking issue (respond within 4 hours)
- **[WARNING]**: Medium priority (respond within 24 hours)
- **[REQUEST]**: Medium priority (respond within 48 hours)
- **[INFO]**: Low priority (acknowledge when convenient)
- **[TEST]**: Low priority (respond immediately and close)

### 3. Investigate the Issue

Example for service issue:

```bash
# Check service status
podman ps -a | grep embedding

# Check service health
curl http://localhost:8080/health

# Test the reported endpoint
curl -X POST http://localhost:8000/api/search \
  -H "Content-Type: application/json" \
  -d '{"library_id": "5673253", "query": "test", "top_k": 1}'
```

### 4. Respond to Sender

**Option A: Send new message back**:
```bash
~/.claude/scripts/send-agent-message.sh cite-assist pin-citer \
  "[RESOLVED] Service issue resolved" \
  "## Issue Resolution

Your message about [issue] has been resolved.

## What I Found
[Root cause]

## Current Status
✅ Service: Healthy
✅ Tests: Passing

## Verification
[Test results]

## Next Steps
[What sender should do]"
```

**Option B: Comment on their issue** (if they created it in your repo):
```bash
gh issue comment <number> --repo your-repo \
  -b "✅ RESOLVED - Details: [explanation]"
```

### 5. Close Original Message

```bash
gh issue close <number> --repo your-repo \
  -c "Handled by your-agent. [summary]"
```

### 6. Mark Todo Complete

```
TodoWrite:
- Check message #381: [ERROR] Embedding service [COMPLETED]
```

---

## Message Types & Examples

### Error Reports (Priority: HIGH)

**From pin-citer to cite-assist**:
```bash
~/.claude/scripts/send-agent-message.sh pin-citer cite-assist \
  "[ERROR] Embedding timeout" \
  "Search queries timing out. Query: 'prosecutorial discretion'. Timeout: 30s. Expected: <5s. Impact: 50% failing."
```

**Response template**:
```bash
~/.claude/scripts/send-agent-message.sh cite-assist pin-citer \
  "[RESOLVED] Embedding timeout fixed" \
  "## Issue Resolution
Embedding service was restarting.

## Current Status
✅ Service: Healthy
✅ Tested: Query returns in 2.3s

## Verification
\`\`\`
curl test shows:
- Response time: 2.3s
- Metadata: Complete
- Score: 0.828
\`\`\`

## Next Steps
Re-run your pipeline."
```

### Configuration Requests (Priority: MEDIUM)

**From pin-citer to cite-assist**:
```bash
~/.claude/scripts/send-agent-message.sh pin-citer cite-assist \
  "[REQUEST] Increase timeout to 60s" \
  "Complex queries need more time. Current 30s timeout is insufficient for multi-term searches."
```

### Status Updates (Priority: LOW)

**From cite-assist to pin-citer**:
```bash
~/.claude/scripts/send-agent-message.sh cite-assist pin-citer \
  "[INFO] Deployment complete" \
  "cite-assist v2.1.0 deployed. Changes: Updated scoring algorithm, improved timeout handling."
```

### Compliance Alerts (Priority: MEDIUM)

**From 12-factor-agents to cite-assist**:
```bash
~/.claude/scripts/send-agent-message.sh 12-factor-agents cite-assist \
  "[WARNING] 12-factor violation" \
  "Config in code detected at src/config.ts:12. Violates Factor #3. Recommend: Move to environment variable DB_URL."
```

### Test Messages (Priority: LOW)

```bash
~/.claude/scripts/send-agent-message.sh test cite-assist \
  "[TEST] Hook verification" \
  "Testing agent messaging system at $(date)"

# Response:
gh issue close <number> -c "Test confirmed. Hooks working."
```

---

## Best Practices

### Message Quality

**DO**:
- ✅ Use clear category prefixes: `[ERROR]`, `[WARNING]`, `[REQUEST]`, `[INFO]`, `[TEST]`
- ✅ Include specific details (endpoints, error messages, timings, impact)
- ✅ Provide verification (test results, status checks)
- ✅ Give clear next steps

**DON'T**:
- ❌ Send vague messages: "Something's wrong"
- ❌ Omit critical context
- ❌ Include sensitive data (credentials, API keys)

### Response Times

- **[ERROR]**: Within 4 hours
- **[WARNING]**: Within 24 hours
- **[REQUEST]**: Within 48 hours
- **[INFO]**: When convenient
- **[TEST]**: Immediately

### Todo List Discipline

1. Add messages immediately when seen
2. Mark `in_progress` when starting investigation
3. Mark `completed` only after:
   - Issue investigated
   - Response sent
   - Original message closed

### Response Format

Use clear, structured responses:

```markdown
## Issue Resolution
[One-line summary]

## What I Found
[Root cause with specifics]

## Current Status
[Verified current state]

## Verification
[Test results with commands]

## Next Steps
[What sender should do]

## Prevention
[How to avoid/monitor]
```

---

## Troubleshooting

### Messages Not Appearing

**Check labels exist**:
```bash
gh label list --repo your-repo | grep agent-message
```

**Verify issue was created**:
```bash
gh issue list --label "agent-message,to:your-agent" --repo your-repo
```

**Test check script**:
```bash
~/.claude/scripts/check-agent-messages.sh your-agent
```

### Can't Send Messages

**Verify script is executable**:
```bash
ls -la ~/.claude/scripts/send-agent-message.sh
```

**Check GitHub CLI auth**:
```bash
gh auth status
```

**Verify target repo labels**:
```bash
gh label list --repo target-repo | grep agent-message
```

### Hooks Not Running

**Verify hooks configuration**:
```bash
cat .claude/hooks-agent-messaging.json
```

**Test script manually**:
```bash
bash ~/.claude/scripts/check-agent-messages.sh your-agent
```

---

## Quick Reference

### Commands

```bash
# Check for messages
~/.claude/scripts/check-agent-messages.sh <your-agent-id>

# Send message
~/.claude/scripts/send-agent-message.sh <from> <to> <title> <message>

# Send with file
~/.claude/scripts/send-agent-message.sh <from> <to> <title> --body-file <file>

# Comment on issue
gh issue comment <number> --repo <repo> -b "comment"

# Close issue
gh issue close <number> --repo <repo> -c "resolution"

# List open messages
gh issue list --label "agent-message,to:<agent>" --repo <repo>
```

### Agent Repository Map

| Agent | Agent ID | Repository |
|-------|----------|------------|
| cite-assist | `cite-assist` | `donaldbraman/cite-assist` |
| pin-citer | `pin-citer` | `donaldbraman/pin-citer` |
| 12-factor-agents | `12-factor-agents` | `donaldbraman/12-factor-agents` |

### Message Categories

| Prefix | Priority | Response Time |
|--------|----------|---------------|
| `[ERROR]` | High | 4 hours |
| `[WARNING]` | Medium | 24 hours |
| `[REQUEST]` | Medium | 48 hours |
| `[INFO]` | Low | When convenient |
| `[TEST]` | Low | Immediate |
| `[BROADCAST]` | Varies | Per message |

---

## Setup Checklist

Use this checklist when setting up a new agent:

- [ ] Install scripts in `~/.claude/scripts/`
- [ ] Make scripts executable (`chmod +x`)
- [ ] Create `agent-message` label in your repo
- [ ] Create `to:<your-agent>` label in your repo
- [ ] Create `from:*` labels for other agents
- [ ] Create `.claude/hooks-agent-messaging.json`
- [ ] Update agent ID in hooks configuration
- [ ] Test send message to yourself
- [ ] Test receive message
- [ ] Test hooks (restart Claude Code)
- [ ] Clean up test messages

---

**GitHub Repository**: https://github.com/donaldbraman/cite-assist
**Reference Implementation**: cite-assist (hooks + scripts)
**Version**: 2.0 (Consolidated Guide)
**Last Updated**: 2025-10-10

---

**Questions or issues?** Create an issue in the cite-assist repository with label `agent-message-system`.
