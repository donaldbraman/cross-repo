# Code Versioning Guide

**Version:** 1.0.1
**Last Updated:** 2025-01-20
**Applies to:** All repositories

## Semantic Versioning

All projects follow [Semantic Versioning 2.0.0](https://semver.org/).

## Version Format

```
MAJOR.MINOR.PATCH
```

**Example:** `2.1.3`

### MAJOR Version (Breaking Changes)

Increment when making incompatible API changes.

**Examples:**

- Removing public API endpoints
- Changing required parameters
- Altering response formats (non-backward compatible)
- Removing or renaming public functions/classes

**Upgrade Path:**

- Document breaking changes
- Provide migration guide
- Consider deprecation period

### MINOR Version (New Features)

Increment when adding functionality in a backward-compatible manner.

**Examples:**

- Adding new API endpoints
- Adding optional parameters
- Adding new public functions/classes
- Enhancing existing features (backward compatible)

**Notes:**

- Reset PATCH to 0
- Previous MINOR versions still work

### PATCH Version (Bug Fixes)

Increment when making backward-compatible bug fixes.

**Examples:**

- Fixing bugs
- Performance improvements
- Documentation updates
- Internal refactoring

**Notes:**

- No API changes
- No new features
- Safe to update immediately

## Version Location

### Python Projects (pyproject.toml)

```toml
[project]
name = "project-name"
version = "1.2.3"
```

### API Versions

```python
# In API code
VERSION = "3.0.0"

# In API response
{
  "api_version": "3.0.0",
  "metadata": {...}
}
```

### Documentation

```markdown
# In guides and docs
**Version:** 1.2.3
**Last Updated:** 2025-10-14
```

## Version Bumping Rules

### When to Bump

**On merge to main:**

- ✅ MAJOR: Breaking changes
- ✅ MINOR: New features
- ✅ PATCH: Bug fixes only

**Not on:**

- ❌ Feature branches
- ❌ WIP commits
- ❌ Documentation-only changes (unless publishing docs)

### How to Bump

```bash
# Example: Bumping from 0.1.0 to 0.2.0 (new feature)

# Edit pyproject.toml
version = "0.2.0"

# Commit
git add pyproject.toml
git commit -m "chore: Bump version to 0.2.0 (v3 API release)"
git push
```

## API Versioning

### Versioned Endpoints

```
/api/v1/search    # Version 1
/api/v2/search    # Version 2 (CSL-compliant)
/api/v3/search    # Version 3 (hybrid search)
```

### Version Strategy

- **Coexistence:** Multiple versions run simultaneously
- **Deprecation:** Announce 6-12 months before removal
- **Documentation:** Each version fully documented

### Version Lifecycle

1. **Launch:** New version released
2. **Stable:** Version mature, most users migrated
3. **Deprecated:** Announced for removal, still functional
4. **Removed:** Version no longer available

## Guide Versioning

### Version Header

Every guide must have a version header:

```markdown
# Guide Title

**Version:** 1.0.0
**Last Updated:** 2025-11-10
**Applies to:** All repositories
```

### When to Bump Guide Versions

**MAJOR version (X.0.0)** - Workflow changes that affect existing usage:

- Changing from merge to squash strategy
- Removing a required step
- Changing command syntax
- Example: `1.2.0 → 2.0.0`

**MINOR version (x.Y.0)** - Adding new content without breaking existing:

- Added new section
- Added new examples
- Added new troubleshooting
- Example: `1.0.0 → 1.1.0`

**PATCH version (x.y.Z)** - Fixes and clarifications:

- Fixed typos
- Clarified existing content
- Updated dates
- Example: `1.0.0 → 1.0.1`

### Changelog Format

Add a changelog section at the end of guides when making significant changes. Follow the [Keep a Changelog](https://keepachangelog.com/) format:

**Standard Sections:**

- **Added:** New features
- **Changed:** Changes in existing functionality
- **Deprecated:** Soon-to-be removed features
- **Removed:** Removed features
- **Fixed:** Bug fixes
- **Security:** Security vulnerabilities

**Simple Format (for guides):**

```markdown
## Changelog

### [1.1.0] - 2025-11-10
- Added: Project-specific details section
- Added: Related guides footer
- Fixed: Clarified label creation workflow

### [1.0.0] - 2025-10-14
- Initial release
```

**Detailed Format (for projects/APIs):**

```markdown
## Changelog

### [2.0.0] - 2025-10-14

#### Added
- v3 API with hybrid search
- Client-controlled weighting

#### Changed
- Default min_score from 0.7 to 0.3

#### Fixed
- Missing title metadata handling

### [1.0.0] - 2025-10-01

#### Added
- Initial release
- v1 and v2 APIs
```

**When to add changelog:**

- ✅ MAJOR or MINOR version bumps
- ❌ PATCH bumps (too granular)

**Where to add:**

- At end of guide, before final "Last updated" footer
- Or in separate CHANGELOG.md file for projects

### Guide Versioning Example

```markdown
# GitHub Labels

**Version:** 1.1.0  ← Bumped from 1.0.0
**Last Updated:** 2025-11-10  ← Updated date
**Applies to:** All repositories

[... guide content ...]

## Changelog

### [1.1.0] - 2025-11-10
- Added: "Always create before using" workflow
- Added: Troubleshooting section
- Added: Project-specific labels section

### [1.0.0] - 2025-10-14
- Initial release

---

*Last updated: 2025-11-10*
```

## Version Communication

### In README

```markdown
## Current Version

**cite-assist:** v2.0.0
**API v3:** Available
**API v2:** Stable
**API v1:** Active
```

### In Agent Messages

```markdown
**Release Version:** cite-assist v0.2.0
```

### In PR Titles

```
chore: Bump version to 2.0.0 (v3 API release)
```

## Best Practices

### Version Planning

1. ✅ Plan breaking changes together
2. ✅ Batch minor features when possible
3. ✅ Document version in issues/PRs
4. ✅ Update CHANGELOG.md

### Version Testing

1. ✅ Test version compatibility
2. ✅ Verify guides reference correct versions
3. ✅ Check API version endpoints
4. ✅ Validate migration paths

### Version Documentation

1. ✅ Document version in all guides
2. ✅ Reference compatibility requirements
3. ✅ Link to CHANGELOG
4. ✅ Provide migration guides

## Common Patterns

### Initial Release

`0.1.0` → Development phase
`1.0.0` → First stable release

### Feature Addition

`1.0.0` → `1.1.0` → `1.2.0`

### Breaking Change

`1.2.0` → `2.0.0`

### Bug Fix

`2.0.0` → `2.0.1` → `2.0.2`

## Project-Specific Details

This is a **global guide** applicable to all repositories.

Check `docs/guides/code-versioning.md` in current repo for:

- Project-specific version locations
- Custom versioning schemes
- Release process details
- Version documentation requirements

## Related Guides

- [Git Workflow](git-workflow.md) - Version bumping workflow and commits

---

*This guide ensures consistent versioning across all projects.*
