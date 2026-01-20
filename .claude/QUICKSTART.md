# Quick Start Guide

**Version:** 1.0.0
**Last Updated:** 2025-01-20

Get started with Claude Code configuration in 5 minutes.

---

## What is the .claude/ Directory?

This directory contains shared resources for Claude Code that work across multiple repositories:

- **Commands** - Slash commands like `/dev-cycle` and `/proof`
- **Guides** - Detailed workflow documentation
- **Agents** - Reusable agent templates for specialized tasks
- **Skills** - Living knowledge documents that improve over time
- **Examples** - Step-by-step walkthroughs of complex workflows

---

## Directory Structure

```
.claude/
├── QUICKSTART.md         # You are here
├── CONFIGURATION.md      # Configuration options reference
├── TROUBLESHOOTING.md    # Common errors and solutions
├── GLOSSARY.md           # Terminology definitions
├── commands/             # Slash commands (/dev-cycle, /proof, etc.)
├── guides/               # Detailed workflow guides
├── agents/               # Reusable agent templates
├── skills/               # Living knowledge documents
├── examples/             # Worked examples
└── templates/            # Schema templates
```

---

## 5-Minute Setup

### 1. Import Shared Resources

Add to your project's `CLAUDE.md`:

```markdown
## Shared Commands
@../cross-repo/.claude/commands/README.md

## Shared Guides
@../cross-repo/.claude/guides/autonomous-cycle.md
```

### 2. Try Your First Command

Run the development cycle for a simple task:

```
/dev-cycle Add a comment to the main function
```

This executes the full autonomous workflow: analyze, create issue, develop, lint, commit, and merge.

### 3. Explore Available Commands

| Command | What It Does |
|---------|--------------|
| `/dev-cycle` | Full autonomous development cycle |
| `/push` | Quick commit and push |
| `/proof` | Proofread documents |
| `/fact-check` | Verify empirical claims |
| `/refactor` | Analyze refactoring opportunities |

---

## Common First Tasks

### Run Your First /dev-cycle

```
/dev-cycle Fix typo in README
```

See [autonomous-cycle-example.md](examples/autonomous-cycle-example.md) for a complete walkthrough.

### Configure Fact-Checking

Add to your project's `CLAUDE.md`:

```markdown
## Fact-Check Configuration

### Verification Tools
- **Domain sources:** site:your-domain.com

### Claim Categories
Standard: Statistics, Study findings, Citations, URLs
```

See [CONFIGURATION.md](CONFIGURATION.md) for all options.

### Proofread a Document

```
/proof path/to/document.pdf
```

See [correction-workflow-example.md](examples/correction-workflow-example.md) for details.

---

## Key Resources

| Resource | Purpose |
|----------|---------|
| [commands/README.md](commands/README.md) | Command reference and usage |
| [guides/autonomous-cycle.md](guides/autonomous-cycle.md) | Worktree-based development guide |
| [agents/README.md](agents/README.md) | Agent templates reference |
| [skills/README.md](skills/README.md) | Living knowledge documents |
| [examples/README.md](examples/README.md) | Worked examples index |
| [CONFIGURATION.md](CONFIGURATION.md) | All configuration options |
| [TROUBLESHOOTING.md](TROUBLESHOOTING.md) | Error solutions |
| [GLOSSARY.md](GLOSSARY.md) | Terminology definitions |

---

## Critical Rules

When using the autonomous development cycle:

- **NEVER** use `--no-verify` to bypass hooks
- **NEVER** work in the main repo directory (use worktrees)
- **ALWAYS** fix lint/hook failures by fixing code
- **ALWAYS** read guide docs before first use

---

## Next Steps

1. **First time?** Read [autonomous-cycle.md](guides/autonomous-cycle.md)
2. **Need examples?** Browse [examples/](examples/)
3. **Having issues?** Check [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
4. **Want configuration options?** See [CONFIGURATION.md](CONFIGURATION.md)

---

*Get productive fast, then explore the details.*
