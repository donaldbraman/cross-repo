# Rate Limit Diagnostics Skill

**Version:** 2.0.0
**Created:** 2024-12-19
**Last Updated:** 2025-01-19

## Purpose

Investigate and diagnose API rate limit issues across various APIs (Gemini, OpenAI, HeinOnline, Google Docs, etc.).

## Trigger

- 429 errors (Too Many Requests)
- RESOURCE_EXHAUSTED errors
- Quota exceeded warnings
- API throttling
- "Rate limit" mentions in error messages

---

## How to Learn (MOST IMPORTANT)

When you diagnose rate limit issues:

1. **Gather context** - Which API, which endpoint, what time of day?
2. **Check actual limits** - Query the API tier and current usage
3. **Identify the pattern** - Burst vs sustained, per-minute vs per-day
4. **Update this skill** - If you learn something new, ADD IT immediately

**Never hesitate to update this file. Living documentation > stale documentation.**

---

## General Diagnostic Steps

### Step 1: Identify the API

Determine which API is rate limited:
- Check error messages for API identifiers
- Review recent API calls in logs
- Check which services are active

### Step 2: Check Rate Limit Headers

Most APIs return rate limit info in response headers:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1640000000
Retry-After: 60
```

### Step 3: Identify the Pattern

**Burst Pattern:**
- Multiple 429 errors in quick succession
- Happens at operation start
- Cause: Too many requests too quickly

**Sustained Pattern:**
- Rate limits appear after running for a while
- Cause: Hitting per-minute or per-hour limits

**Concurrent Pattern:**
- Rate limits when multiple processes run
- Cause: Shared quota exhaustion

### Step 4: Check for Multiple Processes

```bash
# Check for duplicate processes
ps aux | grep "your_script" | grep -v grep
```

---

## API-Specific Reference

### Gemini API (Google AI)

**Tier 2 (Paid) Limits:**
| Model | RPM | TPM | RPD |
|-------|-----|-----|-----|
| gemini-2.0-flash | 4000 | 4M | Unlimited |
| gemini-1.5-flash | 4000 | 4M | Unlimited |
| gemini-embedding-001 | 1500 | 1M | Unlimited |

**Batch API Limits:**
- Max concurrent batch jobs: 50
- Max texts per batch: 100
- Rate limit on submissions: ~1000/min

**Common Errors:**
- `429 RESOURCE_EXHAUSTED` - Hit RPM or TPM limit
- `Quota exceeded` - Daily limit (rare on Tier 2)

**Check Usage:**
- https://ai.dev/usage?tab=rate-limit
- https://console.cloud.google.com/apis/api/generativelanguage.googleapis.com/quotas

### OpenAI API

**Tier Limits (vary by tier):**
- RPM: 500-10,000 depending on tier
- TPM: 40K-2M depending on tier

**Common Errors:**
- `429 Rate limit reached`
- `insufficient_quota`

**Check Usage:**
- https://platform.openai.com/usage

### GitHub API

**Authenticated Limits:**
- 5,000 requests per hour
- Search API: 30 requests per minute

**Check Rate Limit:**
```bash
gh api rate_limit
```

### Google Docs API

**Default Limits:**
- Read: 300 requests per minute per user
- Write: 60 requests per minute per user

---

## Common Patterns and Remediation

### Pattern 1: Burst Submission Exhaustion

**Signature:**
- Multiple 429 errors in quick succession
- Errors on batch job submissions
- Happens at start

**Remediation:**
```python
# Add delay between submissions
await asyncio.sleep(0.2)

# Retry with backoff on 429
if "429" in error_str:
    await asyncio.sleep(5.0)
```

### Pattern 2: Concurrent Process Exhaustion

**Signature:**
- Rate limits appear while running
- Multiple processes detected
- Shared quota exhausted

**Remediation:**
```bash
# Kill duplicate runs
pkill -f "your_script"
# Restart single instance
```

### Pattern 3: Token Limit Exhaustion

**Signature:**
- 429 despite low request count
- Large payloads being sent
- TPM (tokens per minute) error messages

**Remediation:**
- Reduce batch sizes
- Chunk large documents
- Add delays between large requests

---

## Diagnostic Commands

```bash
# Check for multiple processes
ps aux | grep "your_script" | grep -v grep

# Check recent rate limit errors in logs
grep -r "429\|rate\|limit\|RESOURCE_EXHAUSTED" logs/*.log | tail -20

# GitHub rate limit status
gh api rate_limit

# Check system time (for rate limit reset calculations)
date -u
```

---

## Emergency Procedures

### If Rate Limited During Critical Processing

1. **Stop current process** - Prevent further 429s
2. **Wait for reset** - Usually 60 seconds for per-minute limits
3. **Restart with lower concurrency** - Reduce workers/batch size
4. **Monitor closely** - Watch for continued 429s

### If Daily Quota Hit

1. Wait until quota reset (usually midnight in API timezone)
2. Or switch to alternative API/service temporarily
3. Consider upgrading API tier

---

## Adding New API Documentation

When you encounter a new API with rate limits:

1. **Document the limits** - RPM, TPM, RPD, etc.
2. **Document the headers** - How rate limit info is returned
3. **Document common errors** - Exact error messages
4. **Document remediation** - What works for this API
5. **Add usage check URL** - Where to verify current usage

---

*Last updated: 2025-01-19*
