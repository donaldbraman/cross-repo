# Cross-Repository Settings & Documentation

**Purpose**: Central repository for global Claude Code instructions and settings shared across all Donald Braman's agent repositories.

**Status**: Active
**Last Updated**: 2025-10-14

---

## Overview

This repository contains global Claude Code instructions used by all agent projects through symlinks:

- **cite-assist**: Academic citation management and search
- **pin-citer**: Citation pinning and reference management
- **12-factor-agents**: 12-factor app compliance monitoring
- And other related projects

## Contents

### Global Instructions

- **[CLAUDE.md](CLAUDE.md)**: Master copy of global Claude Code instructions
  - Symlinked to `~/CLAUDE.md` and `~/.claude/CLAUDE.md`
  - Contains patterns that apply to all repositories

### Global Guides

Located in `.claude/guides/` (symlinked to `~/.claude/guides/`):

- **[autonomous-cycle.md](.claude/guides/autonomous-cycle.md)**: 8-step autonomous development cycle
- **[agent-messaging.md](.claude/guides/agent-messaging.md)**: Agent-to-agent communication protocol
- **[github-labels.md](.claude/guides/github-labels.md)**: GitHub label conventions
- **[pr-guidelines.md](.claude/guides/pr-guidelines.md)**: Pull request creation standards
- **[lint-and-hooks.md](.claude/guides/lint-and-hooks.md)**: Pre-commit hook handling
- **[git-workflow.md](.claude/guides/git-workflow.md)**: Standard git practices
- **[code-versioning.md](.claude/guides/code-versioning.md)**: Semantic versioning patterns

### Documentation

- **[Agent Messaging Guide](docs/guides/agent-messaging-complete-guide.md)**: Complete implementation guide for agent communication

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

Each project has its own `CLAUDE.md` with **project-specific** content only.

**Example project CLAUDE.md:**
```markdown
# Claude Code Instructions for cite-assist

**Note:** Global instructions are in ~/CLAUDE.md (symlinked from cross-repo)

## Quick Reference

**Active Production Scripts: 16**

## Just-In-Time Guides

[Project-specific guides here...]
```

### For Claude Code Agents

When reading instructions:

1. **Global patterns**: Read from `~/CLAUDE.md` (symlinked to cross-repo)
2. **Project-specific**: Read from project's `CLAUDE.md` or `docs/CLAUDE.md`
3. **Guides**: Access from `~/.claude/guides/` (symlinked to cross-repo)

## Repository Structure

```
cross-repo/
├── README.md                                    # This file
├── CLAUDE.md                                    # Global Claude Code instructions
├── .claude/
│   └── guides/                                  # Global guides (symlinked to ~/.claude/guides)
│       ├── autonomous-cycle.md
│       ├── agent-messaging.md
│       ├── github-labels.md
│       ├── pr-guidelines.md
│       ├── lint-and-hooks.md
│       ├── git-workflow.md
│       └── code-versioning.md
└── docs/
    └── guides/
        └── agent-messaging-complete-guide.md   # Detailed implementation guide
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

*Last updated: 2025-10-10*
