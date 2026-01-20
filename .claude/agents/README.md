# Shared Agent Templates

**Version:** 2.1.0
**Last Updated:** 2025-01-20

## Overview

This directory contains reusable agent templates for Claude Code. These agents are designed to be imported and used across multiple repositories.

## How to Use

### Import in Project CLAUDE.md

Reference these agents in your project's CLAUDE.md file:

```markdown
## Shared Agents
@../cross-repo/.claude/agents/testing-agent.md
@../cross-repo/.claude/agents/janitor-agent.md
@../cross-repo/.claude/agents/code-review-agent.md
@../cross-repo/.claude/agents/documentation-agent.md
```

### Invoke with Task Tool

```
Task tool with:
- description: "[Short description]"
- subagent_type: "general-purpose"
- prompt: <agent template with parameters filled in>
```

## Available Agents

| Agent | Purpose | Version |
|-------|---------|---------|
| [testing-agent](testing-agent.md) | Autonomous test execution with failure/warning reporting | 2.1.0 |
| [janitor-agent](janitor-agent.md) | Repository hygiene maintenance and cleanup | 2.1.0 |
| [code-review-agent](code-review-agent.md) | Code review for PRs, commits, and changes | 1.0.0 |
| [documentation-agent](documentation-agent.md) | Documentation generation, updates, and validation | 1.0.0 |

## Agent Template Structure

All agents follow this standard structure:

```markdown
# Agent Name

**Version:** X.Y.Z
**Created:** YYYY-MM-DD
**Last Updated:** YYYY-MM-DD
**Agent Type:** general-purpose|Bash|Explore|Plan

## Purpose
[What the agent does]

## Input Contract
[Required and optional parameters]

## Output Contract
[Expected output format]

## Prompt Template
[The actual prompt to use with the Task tool]

## Version History
[Changes by version]

## Success Metrics
[How to measure agent effectiveness]

## Usage Examples
[How to invoke the agent]

## Validation Checklist
[How to verify agent works correctly]
```

## Versioning

Agents use semantic versioning:
- **Patch (X.Y.Z):** Bug fixes, typo corrections
- **Minor (X.Y.0):** New features, backwards-compatible changes
- **Major (X.0.0):** Breaking changes to input/output contracts

## Contributing

To add a new shared agent:

1. Create the agent in this directory following the template structure
2. Add an entry to this README
3. Test in at least one project
4. Document usage examples

## Project-Specific Overrides

If a project needs to customize a shared agent, it can:

1. Import the base agent
2. Add project-specific sections in the project's CLAUDE.md
3. Or create a local agent that extends the shared one

---

*Shared agents reduce duplication and ensure consistent patterns across repositories.*
