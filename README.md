# Cross-Repository Settings & Documentation

**Purpose**: Central repository for settings, guides, and documentation shared across all Donald Braman's Claude Code agent repositories.

**Status**: Active
**Last Updated**: 2025-10-10

---

## Overview

This repository contains cross-repository resources used by multiple Claude Code agent projects:

- **cite-assist**: Academic citation management and search
- **pin-citer**: Citation pinning and reference management
- **12-factor-agents**: 12-factor app compliance monitoring
- And other related projects

## Contents

### Documentation

- **[Agent Messaging Guide](docs/guides/agent-messaging-complete-guide.md)**: Complete guide for agent-to-agent communication via GitHub issues
  - Setup instructions
  - Sending/receiving messages
  - Message handling workflows
  - Best practices and troubleshooting

### Future Additions

This repository may include:

- Shared scripts and utilities
- Common configuration templates
- Cross-project development guidelines
- Shared hooks configurations
- Integration patterns and examples

## Usage

### For Repository Maintainers

Reference documentation from this repository in your project-specific `CLAUDE.md`:

```markdown
## Agent Messaging

See the complete guide: [cross-repo/docs/guides/agent-messaging-complete-guide.md](https://github.com/donaldbraman/cross-repo/blob/main/docs/guides/agent-messaging-complete-guide.md)
```

### For Claude Code Agents

When working across multiple repositories, agents should:

1. Check this repository for cross-cutting guidance
2. Follow repository-specific `CLAUDE.md` for project details
3. Use standardized patterns documented here

## Repository Structure

```
cross-repo/
├── README.md                                    # This file
├── docs/
│   └── guides/
│       └── agent-messaging-complete-guide.md   # Agent communication guide
└── [future additions]
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
