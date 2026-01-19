# Cross-Repository Settings & Documentation

**Purpose**: Central repository for shared Claude Code instructions, agents, skills, commands, and guides across all Donald Braman's repositories.

**Status**: Active
**Last Updated**: 2025-01-19

---

## Overview

This repository serves as a **shared library** for Claude Code resources used across multiple projects:

- **cite-assist**: Academic citation management and search
- **write-assist**: Academic writing assistance
- **pin-citer**: Citation pinning and reference management
- **zothein**: Zotero-HeinOnline integration
- **criminal-law**: Legal research and writing
- **chirho**: Causal inference research
- And other related projects

## Sharing Approach

### Recommended: Import Syntax

Projects import shared resources using relative paths in their CLAUDE.md:

```markdown
## Shared Resources
@../cross-repo/.claude/guides/worktree-dev-cycle.md
@../cross-repo/.claude/agents/testing-agent.md
@../cross-repo/.claude/skills/rate-limit-diagnostics/SKILL.md
```

**Benefits:**
- Native Claude Code feature
- Explicit about what's imported
- No symlink cross-platform issues
- Works with relative paths

### Legacy: Symlinks

Some projects still use symlinks (gradually migrating to imports):
- `~/.claude/guides/` symlinked to cross-repo guides
- Project `.claude/guides/` symlinked to cross-repo

## Contents

### Global Instructions

- **[CLAUDE.md](CLAUDE.md)**: Master copy of global Claude Code instructions
  - Symlinked to `~/CLAUDE.md` and `~/.claude/CLAUDE.md`
  - Contains patterns that apply to all repositories

### Shared Agents

Located in `.claude/agents/`:

| Agent | Purpose | Version |
|-------|---------|---------|
| **[testing-agent](.claude/agents/testing-agent.md)** | Autonomous test execution with failure/warning reporting | 2.0.0 |
| **[janitor-agent](.claude/agents/janitor-agent.md)** | Repository hygiene maintenance and cleanup | 2.0.0 |

See [Agents README](.claude/agents/README.md) for usage.

### Shared Skills

Located in `.claude/skills/`:

| Skill | Purpose | Version |
|-------|---------|---------|
| **[rate-limit-diagnostics](.claude/skills/rate-limit-diagnostics/SKILL.md)** | Diagnose API rate limit issues (Gemini, OpenAI, etc.) | 2.0.0 |

See [Skills README](.claude/skills/README.md) for usage.

### Shared Commands

Located in `.claude/commands/`:

| Command | Purpose | Version |
|---------|---------|---------|
| **[fact-check](.claude/commands/fact-check.md)** | Verify empirical claims, statistics, and citations | 1.0.0 |
| **[proof](.claude/commands/proof.md)** | Proofread documents (PDF, Markdown, text) | 2.1.0 |

See [Commands README](.claude/commands/README.md) for usage.

### Shared Guides

Located in `.claude/guides/`:

- **[autonomous-cycle.md](.claude/guides/autonomous-cycle.md)**: 6-step worktree-based autonomous development cycle
- **[agent-messaging.md](.claude/guides/agent-messaging.md)**: Agent-to-agent communication protocol
- **[github-labels.md](.claude/guides/github-labels.md)**: GitHub label conventions
- **[pr-guidelines.md](.claude/guides/pr-guidelines.md)**: Pull request creation standards
- **[lint-and-hooks.md](.claude/guides/lint-and-hooks.md)**: Pre-commit hook handling
- **[git-workflow.md](.claude/guides/git-workflow.md)**: Standard git practices
- **[code-versioning.md](.claude/guides/code-versioning.md)**: Semantic versioning patterns

## Setup

### First-Time Setup on New Machine

1. Clone cross-repo:
```bash
cd ~/Documents/GitHub
git clone https://github.com/donaldbraman/cross-repo.git
```

2. Create symlinks:
```bash
# Create .claude directory if needed
mkdir -p ~/.claude

# Symlink CLAUDE.md
ln -sf ~/Documents/GitHub/cross-repo/CLAUDE.md ~/CLAUDE.md
ln -sf ~/Documents/GitHub/cross-repo/CLAUDE.md ~/.claude/CLAUDE.md

# Symlink guides directory
ln -sf ~/Documents/GitHub/cross-repo/.claude/guides ~/.claude/guides
```

3. Verify symlinks:
```bash
# Should show cross-repo content
cat ~/CLAUDE.md | head -5

# Should show "symbolic link"
ls -la ~/CLAUDE.md
ls -la ~/.claude/
```

### Broken Symlink Recovery

If symlinks break, recreate them:
```bash
rm ~/CLAUDE.md ~/.claude/CLAUDE.md ~/.claude/guides
ln -sf ~/Documents/GitHub/cross-repo/CLAUDE.md ~/CLAUDE.md
ln -sf ~/Documents/GitHub/cross-repo/CLAUDE.md ~/.claude/CLAUDE.md
ln -sf ~/Documents/GitHub/cross-repo/.claude/guides ~/.claude/guides
```

## Usage

### For Repository Maintainers

Each project has its own `CLAUDE.md` that imports shared resources and adds project-specific content.

**Example project CLAUDE.md:**
```markdown
# Claude Code Instructions for my-project

**Note:** Global instructions are in ~/CLAUDE.md (symlinked from cross-repo)

## Shared Resources

Import what you need from cross-repo:

@../cross-repo/.claude/guides/worktree-dev-cycle.md
@../cross-repo/.claude/guides/agent-messaging.md
@../cross-repo/.claude/agents/testing-agent.md
@../cross-repo/.claude/agents/janitor-agent.md
@../cross-repo/.claude/skills/rate-limit-diagnostics/SKILL.md
@../cross-repo/.claude/commands/proof.md

## Project-Specific Content

[Your project-specific guides, agents, and commands here...]
```

### How Imports Work

1. **Path Convention**: All repos are siblings under `~/Documents/GitHub/`
2. **Relative Paths**: Use `@../cross-repo/...` to import
3. **Selective Import**: Only import what your project needs
4. **Local Override**: Project-local files take precedence over imports

### For Claude Code Agents

When reading instructions:

1. **Global patterns**: Read from `~/CLAUDE.md` (symlinked to cross-repo)
2. **Imported resources**: Automatically available via @ imports in project CLAUDE.md
3. **Project-specific**: Read from project's `CLAUDE.md` for local customizations

## Repository Structure

```
cross-repo/
├── README.md                                    # This file
├── CLAUDE.md                                    # Global Claude Code instructions
└── .claude/
    ├── agents/                                  # Shared agent templates
    │   ├── README.md
    │   ├── testing-agent.md
    │   └── janitor-agent.md
    ├── skills/                                  # Shared skills (living docs)
    │   ├── README.md
    │   └── rate-limit-diagnostics/
    │       └── SKILL.md
    ├── commands/                                # Shared commands
    │   ├── README.md
    │   ├── fact-check.md
    │   └── proof.md
    └── guides/                                  # Shared guides
        ├── autonomous-cycle.md
        ├── agent-messaging.md
        ├── github-labels.md
        ├── pr-guidelines.md
        ├── lint-and-hooks.md
        ├── git-workflow.md
        └── code-versioning.md
```

### Symlink Structure

```
~/CLAUDE.md → ~/Documents/GitHub/cross-repo/CLAUDE.md
~/.claude/CLAUDE.md → ~/Documents/GitHub/cross-repo/CLAUDE.md
~/.claude/guides/ → ~/Documents/GitHub/cross-repo/.claude/guides/
```

---

## Contributing

This repository is maintained alongside the individual agent repositories. Updates should:

- Be applicable across multiple repositories
- Follow existing documentation patterns
- Include version dates and status indicators

---

**Maintained by**: Donald Braman
**Related Repositories**:
- [cite-assist](https://github.com/donaldbraman/cite-assist)
- [pin-citer](https://github.com/donaldbraman/pin-citer)
- [12-factor-agents](https://github.com/donaldbraman/12-factor-agents)

---

*Last updated: 2025-01-19*
