# CLAUDE.md Configuration Reference

**Version:** 1.0.0
**Last Updated:** 2025-01-20

This document describes all configuration options that can be set in a project's `CLAUDE.md` file to customize command behavior.

---

## Overview

Commands in this repository read project-specific settings from `CLAUDE.md`. This centralized configuration allows projects to customize:

- Verification tools and data sources
- Style guides and formatting conventions
- Output directories and file locations
- Domain-specific categories and rules

---

## Configuration Sections

### Fact-Check Configuration

Configure the `/fact-check` command for verifying empirical claims.

```markdown
## Fact-Check Configuration

### Verification Tools
- **Local search:** [tool name] at [endpoint]
- **External search:** [tool name] for [database]
- **Domain sources:** site:[domain1], site:[domain2]

### Claim Categories
Standard: Statistics, Study findings, Citations, URLs
Additional: [Domain-specific category 1], [Category 2]

### Report Settings
- **Output directory:** temp/
- **Apply command:** /apply (if available)
```

#### Options

| Option | Description | Default | Affects |
|--------|-------------|---------|---------|
| **Verification Tools - Local** | Local search API endpoint for document verification | None | `/fact-check` Step 4 (Tier 1) |
| **Verification Tools - External** | External academic search tool (e.g., OpenAlex, Semantic Scholar) | None | `/fact-check` Step 4 (Tier 2) |
| **Verification Tools - Domain sources** | Domain-specific search sites for web searches | None | `/fact-check` Step 4 (Tier 3) |
| **Claim Categories - Standard** | Built-in categories always verified | Statistics, Study findings, Citations, URLs | `/fact-check` Step 1 |
| **Claim Categories - Additional** | Project-specific claim types to verify | None | `/fact-check` Step 1 |
| **Report Settings - Output directory** | Directory for correction reports | `temp/` | `/fact-check` Step 6 |
| **Report Settings - Apply command** | Command to apply corrections | `/apply` | `/fact-check` Step 7 |

#### Example: Legal Research Project

```markdown
## Fact-Check Configuration

### Verification Tools
- **Local search:** cite-assist API at localhost:8000
- **External search:** discovery CLI for OpenAlex/Semantic Scholar
- **Domain sources:** site:supremecourt.gov, site:law.cornell.edu

### Claim Categories
Standard: Statistics, Study findings, Citations, URLs
Additional: Legal holdings, Case citations, Statutory references

### Report Settings
- **Output directory:** temp/
```

#### Example: Scientific Paper

```markdown
## Fact-Check Configuration

### Verification Tools
- **Local search:** papers-api at localhost:3000
- **External search:** semantic-scholar-cli

### Claim Categories
Standard: Statistics, Study findings, Citations, URLs
Additional: Experimental results, Technical specifications

### Report Settings
- **Output directory:** reports/
```

---

### Style Guide Configuration

Configure style guides for `/proof` and other formatting commands.

```markdown
## Style Guide

See [project-style-guide.md](docs/project-style-guide.md) for:
- Citation format (e.g., Bluebook, APA, Chicago)
- Terminology standards
- Formatting conventions
```

#### Options

| Option | Description | Default | Affects |
|--------|-------------|---------|---------|
| **Style Guide Link** | Path to project-specific style guide | None | `/proof` Step 1 |
| **Citation Format** | Citation style (Bluebook, APA, Chicago, etc.) | None | `/proof`, `/fact-check` |
| **Terminology Standards** | Project-specific terminology rules | None | `/proof` Step 2 |

#### Example: Academic Project

```markdown
## Style Guide

See [style-guide.md](docs/style-guide.md) for:
- Citation format: APA 7th edition
- Terminology: Use "participants" not "subjects"
- Formatting: Oxford comma required
```

#### Example: Legal Project

```markdown
## Style Guide

See [bluebook-guide.md](docs/bluebook-guide.md) for:
- Citation format: Bluebook 21st edition
- Court name abbreviations
- Signal usage (e.g., see, cf., but see)
```

---

### Report Output Configuration

Configure where commands save their output files.

```markdown
## Report Settings

- **Output directory:** temp/
- **Report format:** markdown
```

#### Options

| Option | Description | Default | Affects |
|--------|-------------|---------|---------|
| **Output directory** | Directory for generated reports | `temp/` | `/fact-check`, `/proof` |
| **Report format** | Format for output reports | `markdown` | All commands with report output |

---

## Full Configuration Example

Here is a complete example showing all configuration options:

```markdown
# Project CLAUDE.md

## Fact-Check Configuration

### Verification Tools
- **Local search:** cite-assist API at localhost:8000
- **External search:** discovery CLI for OpenAlex/Semantic Scholar
- **Domain sources:** site:bls.gov, site:census.gov

### Claim Categories
Standard: Statistics, Study findings, Citations, URLs
Additional: Economic indicators, Policy claims

### Report Settings
- **Output directory:** temp/
- **Apply command:** /apply

---

## Style Guide

See [style-guide.md](docs/style-guide.md) for:
- Citation format: Chicago 17th edition
- Terminology standards
- Formatting conventions

---

## Report Settings

- **Output directory:** temp/
- **Report format:** markdown
```

---

## Command-Specific Defaults

When no configuration is found in CLAUDE.md, commands use these defaults:

| Command | Default Behavior |
|---------|------------------|
| `/fact-check` | Web search only, standard claim categories, reports to `temp/` |
| `/proof` | No style guide, standard proofreading rules |

---

## Related

- [fact-check](commands/fact-check.md) - Verify empirical claims
- [proof](commands/proof.md) - Document proofreading
- [GLOSSARY](GLOSSARY.md) - Terminology definitions

---

## Version History

### 1.0.0 (2025-01-20)

- Initial release
- Documented Fact-Check Configuration options
- Documented Style Guide Configuration options
- Documented Report Output Configuration options
- Added full configuration example
