# Shared Commands

**Version:** 1.0.0
**Last Updated:** 2025-01-19

## Overview

Commands are slash-invokable actions that can be triggered with `/$command-name`. This directory contains commands designed to work across multiple repositories.

## How to Use

### Import in Project CLAUDE.md

Reference commands in your project's CLAUDE.md file:

```markdown
## Shared Commands
@../cross-repo/.claude/commands/proof.md
```

### Invoke Command

Once imported, the command can be invoked with:

```
/proof path/to/document.pdf
```

## Available Commands

| Command | Purpose | Arguments | Version |
|---------|---------|-----------|---------|
| [dev-cycle](dev-cycle.md) | Execute full autonomous development cycle | task description or issue # | 1.0.0 |
| [fact-check](fact-check.md) | Verify empirical claims, statistics, and citations | file path, optional section | 1.0.0 |
| [proof](proof.md) | Proofread documents (PDF, Markdown, text) | file path, optional page range | 2.1.0 |
| [push](push.md) | Stage, commit, and push all changes | optional commit message | 1.0.0 |

## Command Structure

All commands follow this structure:

```markdown
# Command Name

**Version:** X.Y.Z
**Last Updated:** YYYY-MM-DD

[Description of what the command does. $ARGUMENTS represents user input.]

## Instructions

[Detailed instructions for executing the command]

## Version History

[Changes by version]
```

### Arguments

Commands receive user input via `$ARGUMENTS`. For example:

- `/proof document.pdf` -> `$ARGUMENTS` = `document.pdf`
- `/proof document.pdf 1-10` -> `$ARGUMENTS` = `document.pdf 1-10`

## Versioning

Commands use semantic versioning:
- **Patch (X.Y.Z):** Bug fixes, clarifications
- **Minor (X.Y.0):** New options, expanded functionality
- **Major (X.0.0):** Changed invocation syntax, breaking changes

## Contributing

To add a new shared command:

1. Create the command markdown file in this directory
2. Add an entry to this README
3. Test in at least one project
4. Document all supported arguments

## Project-Specific Commands

Projects can have local commands in their `.claude/commands/` directory that:

- Extend shared commands with project-specific behavior
- Handle project-specific operations
- Override shared commands if needed (local takes precedence)

---

*Commands provide quick access to common operations.*
