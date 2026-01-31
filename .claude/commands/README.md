# Shared Commands

**Version:** 1.1.0
**Last Updated:** 2025-01-20

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
| [fact-check](fact-check.md) | Verify empirical claims and apply corrections | file path, --report-only | 3.0.0 |
| [proof](proof.md) | Proofread documents and apply corrections | file path, --report-only | 3.4.0 |
| [merge](merge.md) | Stage, commit, push, review, and merge | optional commit message | 1.0.0 |
| [push](push.md) | Stage, commit, and push all changes | optional commit message | 1.0.0 |
| [refactor](refactor.md) | Analyze codebase for refactoring opportunities | --report, --path, --max, --auto-approve | 1.0.0 |
| [repeat-back](repeat-back.md) | Repeat back user query with clarity to confirm understanding | optional query text | 1.0.0 |

### Companion Files

Some commands are split into modular files for maintainability:

| File | Parent Command | Contents |
|------|----------------|----------|
| [fact-check-methodology](fact-check-methodology.md) | fact-check | Detailed step-by-step verification process |
| [fact-check-reference](fact-check-reference.md) | fact-check | Claim types, red flags, verification patterns |

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

## Reference Documents

Supporting documentation for commands and workflows:

### Core References

| Document | Description |
|----------|-------------|
| [GLOSSARY.md](../GLOSSARY.md) | Central terminology definitions used across all documentation |
| [CONFIGURATION.md](../CONFIGURATION.md) | All CLAUDE.md configuration options for customizing commands |
| [TROUBLESHOOTING.md](../TROUBLESHOOTING.md) | Consolidated troubleshooting hub for common issues |

### Templates

| Template | Description |
|----------|-------------|
| [correction-report-schema.md](../templates/correction-report-schema.md) | Standard schema for correction reports used by `/proof` and `/fact-check` |

### Worked Examples

| Example | Description |
|---------|-------------|
| [examples/](../examples/README.md) | Directory of worked examples demonstrating complex workflows |

### Guides

Detailed workflow guides are available in the [guides/](../guides/) directory:

| Guide | Description |
|-------|-------------|
| [autonomous-cycle.md](../guides/autonomous-cycle.md) | Full development workflow with worktrees |
| [correction-workflow.md](../guides/correction-workflow.md) | Applying corrections from reports |
| [lint-and-hooks.md](../guides/lint-and-hooks.md) | Pre-commit hook handling |
| [pr-guidelines.md](../guides/pr-guidelines.md) | Pull request creation standards |

---

*Commands provide quick access to common operations.*
