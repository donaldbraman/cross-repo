# Shared Skills

**Version:** 1.0.0
**Last Updated:** 2025-01-19

## Overview

Skills are living documents containing domain knowledge that agents use AND update. Unlike static documentation, skills evolve as Claude learns from each invocation.

## How to Use

### Import in Project CLAUDE.md

Reference skills in your project's CLAUDE.md file:

```markdown
## Shared Skills
@../cross-repo/.claude/skills/rate-limit-diagnostics/SKILL.md
```

### Key Principle

**Skills are living documents.** When you diagnose an issue and learn something new, **update the skill immediately**.

## Available Skills

| Skill | Purpose | Version |
|-------|---------|---------|
| [rate-limit-diagnostics](rate-limit-diagnostics/SKILL.md) | Diagnose API rate limit issues | 2.0.0 |

## Skill Structure

Skills use a directory structure:

```
skill-name/
├── SKILL.md          # Main skill document
├── examples/         # Optional: example cases
└── templates/        # Optional: reusable templates
```

### SKILL.md Template

```markdown
# Skill Name

**Version:** X.Y.Z
**Created:** YYYY-MM-DD
**Last Updated:** YYYY-MM-DD

## Purpose
[What this skill helps with]

## Trigger
[When this skill should be activated]

## How to Learn
[MOST IMPORTANT: How to update this skill with new knowledge]

## Knowledge Base
[The actual domain knowledge]

## Diagnostic Steps
[How to investigate issues]

## Common Patterns
[Known patterns and solutions]

## Adding New Knowledge
[How to contribute new information]
```

## Versioning

Skills use semantic versioning:

- **Patch (X.Y.Z):** New examples, typo fixes
- **Minor (X.Y.0):** New patterns, expanded knowledge
- **Major (X.0.0):** Restructured knowledge base, changed diagnostic approach

## Contributing

To add a new shared skill:

1. Create a directory: `skill-name/`
2. Create `SKILL.md` following the template
3. Add an entry to this README
4. Document when the skill should trigger
5. Include "How to Learn" section for self-updating

## Project-Specific Skills

Projects can have local skills that:

- Extend shared skills with project-specific knowledge
- Handle domain-specific issues not applicable elsewhere
- Contain sensitive information not suitable for sharing

---

*Skills grow smarter with every use.*
