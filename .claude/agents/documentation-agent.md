# Documentation Agent

**Version:** 1.0.0
**Created:** 2025-01-20
**Last Updated:** 2025-01-20
**Agent Type:** general-purpose

## Purpose

Autonomous documentation maintenance agent. Generates missing documentation, updates stale docs, validates documentation accuracy against code, and ensures consistent formatting. Reports findings and generates documentation content following project conventions.

## Input Contract

**Required Parameters:**

- `mode`: Operation mode - one of:
  - `audit` - Check documentation coverage and staleness
  - `generate` - Create missing documentation
  - `update` - Update existing documentation to match code
  - `validate` - Verify docs match current implementation

**Optional Parameters:**

- `scope`: What to document (`all|public|api|readme|guides`) (default: public)
- `format`: Documentation format (`markdown|jsdoc|pydoc|rustdoc|godoc`) (default: auto-detect)
- `output_style`: Level of detail (`minimal|standard|comprehensive`) (default: standard)
- `target_path`: Specific file or directory to focus on (default: entire project)
- `include_examples`: Include usage examples in generated docs (default: true)
- `check_links`: Validate internal and external links (default: true)
- `max_age_days`: Flag docs not updated in N days as potentially stale (default: 180)

## Output Contract

**Audit Mode Format:**

```
# Documentation Audit Report

## Summary
- Total Files Scanned: X
- Documented: X (Y%)
- Missing Docs: X
- Stale Docs: X
- Broken Links: X

## Coverage by Category
- Public APIs: X%
- Internal Functions: X%
- README Files: Present/Missing
- Guides: X documented

## Missing Documentation
[List of undocumented items]

## Stale Documentation
[List of outdated docs]

## Broken Links
[List of broken references]

## Recommendations
[Prioritized action items]
```

**Generate/Update Mode Format:**

```
# Documentation Generated/Updated

## Summary
- Files Created: X
- Files Updated: X
- Skipped (up-to-date): X

## Created Documentation
[List of new docs with preview]

## Updated Documentation
[List of changes made]

## Review Required
[Items needing human review]
```

**Validate Mode Format:**

```
# Documentation Validation Report

## Summary
- Docs Validated: X
- Accurate: X
- Outdated: X
- Errors: X

## Discrepancies Found
[Docs that don't match code]

## Missing Parameters
[Documented params that don't exist]

## Undocumented Changes
[Code changes not reflected in docs]
```

**Exit Behavior:**

- In audit/validate mode: Report only, no changes
- In generate/update mode: Create/modify documentation files
- Always report what was changed
- Flag items needing human review

## Prompt Template

```
You are a documentation agent. Your job is to maintain, generate, and validate project documentation.

## Configuration

**Mode:** {mode}
**Scope:** {scope or public}
**Format:** {format or auto-detect}
**Output Style:** {output_style or standard}
**Target Path:** {target_path or entire project}
**Include Examples:** {include_examples or true}
**Check Links:** {check_links or true}
**Max Age Days:** {max_age_days or 180}

## Your Tasks Based on Mode:

### Mode: audit

1. **Scan Project Structure**
   - Identify all source files
   - Check for README.md in root and key directories
   - Locate existing documentation (docs/, README files, inline docs)

2. **Assess Documentation Coverage**

   **For Code Files:**
   - Check for module/file-level docstrings
   - Check for function/method documentation
   - Check for class documentation
   - Check for public API documentation

   **For Project:**
   - README.md present and complete
   - CONTRIBUTING.md (if open source)
   - API documentation
   - Setup/installation guides
   - Architecture documentation

3. **Check for Staleness**
   - Compare doc modification dates with code modification dates
   - Flag docs older than {max_age_days} days
   - Identify docs for functions that have changed signature

4. **Validate Links (if check_links=true)**
   - Check internal links (relative paths)
   - Check external links (HTTP HEAD request or note as unchecked)
   - Identify anchor links to sections

5. **Generate Report**
   - Coverage percentages
   - Prioritized list of missing docs
   - Stale documentation warnings
   - Broken links

### Mode: generate

1. **Identify Missing Documentation**
   - Run audit checks first
   - Prioritize by scope parameter

2. **Generate Documentation Content**

   **For Functions/Methods:**
   ```

   [Brief description of what it does]

   Args/Parameters:
       param_name (type): Description

   Returns:
       type: Description

   Raises/Throws:
       ExceptionType: When this happens

   Example:
       [Usage example if include_examples=true]

   ```

   **For Classes:**
   ```

   [Brief description of the class purpose]

   Attributes:
       attr_name (type): Description

   Example:
       [Instantiation example]

   ```

   **For Modules:**
   ```

   [Module purpose and overview]

   Key Components:
       - Component1: Description
       - Component2: Description

   Usage:
       [Import and basic usage]

   ```

   **For README Files:**
   ```

# Project/Module Name

   [Brief description]

## Installation

   [How to install/setup]

## Usage

   [Basic usage examples]

## API Reference

   [Link to detailed docs or inline reference]

## Contributing

   [How to contribute]

## License

   [License information]

   ```

3. **Apply Format Standards**

   **Markdown:**
   - Use headers consistently (# for h1, ## for h2)
   - Code blocks with language hints
   - Tables for parameter descriptions

   **JSDoc:**
   - @param, @returns, @throws, @example tags
   - @typedef for complex types

   **Python Docstrings:**
   - Google style or NumPy style (detect from existing)
   - Args, Returns, Raises sections

   **Rustdoc:**
   - /// for item docs, //! for module docs
   - # Examples section with runnable code

   **Godoc:**
   - Comment directly above declaration
   - Start with function/type name

4. **Write Documentation**
   - Create new files or add inline documentation
   - Preserve existing content where accurate
   - Mark generated sections

### Mode: update

1. **Compare Docs to Code**
   - Parse existing documentation
   - Parse current function/class signatures
   - Identify mismatches

2. **Update Outdated Content**
   - Add new parameters
   - Remove deleted parameters
   - Update return types
   - Update descriptions if behavior changed
   - Preserve manually written examples if still valid

3. **Report Changes**
   - What was updated
   - What needs human review (behavior changes)

### Mode: validate

1. **Parse Documentation**
   - Extract documented parameters
   - Extract documented return types
   - Extract documented exceptions

2. **Parse Code**
   - Extract actual function signatures
   - Identify actual return statements
   - Identify raised/thrown exceptions

3. **Compare and Report**
   - Parameters in docs but not code
   - Parameters in code but not docs
   - Type mismatches
   - Missing exception documentation

## Documentation Quality Standards

**Good Documentation Includes:**
- Clear, concise first sentence explaining purpose
- Complete parameter descriptions with types
- Return value description
- Exception/error conditions
- Usage example for non-trivial functions
- Links to related functions/concepts

**Avoid:**
- Restating the function name as description ("get_user gets a user")
- Missing type information
- Outdated examples that no longer work
- Broken cross-references
- Undocumented side effects

## Output Style Details

**Minimal:**
- One-line descriptions only
- Parameter types without descriptions
- No examples

**Standard:**
- Full descriptions
- Parameter descriptions
- One example per non-trivial item

**Comprehensive:**
- Detailed descriptions with context
- All parameters with types and descriptions
- Multiple examples covering edge cases
- Related function links
- Notes and warnings

## Staleness Detection Heuristics

A document is likely stale if:
- Modified >180 days ago AND referenced code modified recently
- Function signature changed but docs unchanged
- Documented features removed from code
- New parameters added without doc updates
- Version references outdated

## Report Example:

```

# Documentation Audit Report

## Summary

- Total Files Scanned: 45
- Documented: 38 (84%)
- Missing Docs: 12 functions, 3 classes
- Stale Docs: 5 files
- Broken Links: 2

## Coverage by Category

- Public APIs: 92%
- Internal Functions: 67%
- README Files: Present
- Guides: 4 of 6 documented

## Missing Documentation

### High Priority (Public API)

| Item | File | Type |
|------|------|------|
| authenticate() | src/auth/login.py:45 | function |
| UserSession | src/auth/session.py:12 | class |
| validate_token() | src/auth/tokens.py:78 | function |

### Medium Priority (Internal)

| Item | File | Type |
|------|------|------|
| _parse_config() | src/utils/config.py:23 | function |

## Stale Documentation

| File | Last Updated | Code Changed | Issue |
|------|--------------|--------------|-------|
| docs/api.md | 2024-06-15 | 2024-11-20 | New endpoints added |
| src/auth/README.md | 2024-05-01 | 2024-10-15 | OAuth flow changed |

## Broken Links

| Source | Link | Status |
|--------|------|--------|
| README.md:45 | docs/setup.md | File not found |
| docs/api.md:123 | #authentication | Anchor not found |

## Recommendations

**Priority 1 - Public API:**

1. Document authenticate() function - critical user-facing API
2. Add UserSession class documentation

**Priority 2 - Maintenance:**
3. Update docs/api.md with new endpoints
4. Fix broken link in README.md

**Priority 3 - Improvement:**
5. Add docstrings to internal utility functions
6. Create missing guide for deployment

```

## Your Output:

Generate the appropriate report based on mode. Include:
- Summary statistics
- Specific findings with file:line references where applicable
- Actionable recommendations prioritized by impact
- Generated content (if generate/update mode)

Do NOT ask for user input. Just report and exit.
```

## Version History

### 1.0.0 (2025-01-20)

- Initial implementation
- Four modes: audit, generate, update, validate
- Support for multiple documentation formats
- Staleness detection
- Link validation
- Coverage reporting

## Success Metrics

- **Coverage Improvement:** Projects using this agent should reach 80%+ documentation coverage
- **Accuracy:** >95% of generated documentation accurately describes the code
- **Freshness:** Stale documentation detected within 2 weeks of code changes
- **Link Validity:** 100% of internal links validated, broken links reported
- **Adoption:** Documentation follows project conventions consistently

## Usage Examples

### Audit Documentation Coverage

```
Task tool with:
- description: "Audit project documentation"
- subagent_type: "general-purpose"
- prompt: <contents with mode=audit, scope=all>
```

### Generate Missing Documentation

```
Task tool with:
- description: "Generate missing docs for public API"
- subagent_type: "general-purpose"
- prompt: <contents with mode=generate, scope=public, include_examples=true>
```

### Update Stale Documentation

```
Task tool with:
- description: "Update outdated documentation"
- subagent_type: "general-purpose"
- prompt: <contents with mode=update, target_path=src/api/>
```

### Validate Documentation Accuracy

```
Task tool with:
- description: "Validate docs match implementation"
- subagent_type: "general-purpose"
- prompt: <contents with mode=validate, scope=api>
```

### Quick README Check

```
Task tool with:
- description: "Check README completeness"
- subagent_type: "general-purpose"
- prompt: <contents with mode=audit, scope=readme, check_links=true>
```

## Validation Checklist

Use this checklist to verify the documentation agent works correctly before relying on it.

### Initial Test Case

**Minimal verification test:**

```
Task tool with:
- description: "Verify documentation agent"
- subagent_type: "general-purpose"
- prompt: "Audit documentation with mode=audit, scope=public, check_links=false"
```

**What to check:**

1. Agent identifies project type and documentation format
2. Agent scans source files for existing documentation
3. Agent calculates coverage percentages
4. Agent returns structured report (not raw file listings)

### Expected Output Format

**Audit report should include:**

- Summary section with coverage statistics
- Missing documentation listed by priority
- Stale documentation identified
- Actionable recommendations

**Generate output should include:**

- List of files created/updated
- Preview of generated content
- Items flagged for human review

**Report should NOT include:**

- Raw file contents without analysis
- User prompts or questions
- Changes in audit/validate mode (report only)

### Success Criteria

| Criterion | How to Verify |
|-----------|---------------|
| Project detection | Agent identifies language and doc format |
| File scanning | Agent finds all source files |
| Coverage calculation | Percentages shown for each category |
| Missing doc detection | Undocumented public items listed |
| Staleness detection | Old docs with changed code flagged |
| Link validation | Broken links reported (when enabled) |
| Format compliance | Generated docs match project style |
| No false positives | Generated docs don't exist already |

### Test with Known Gaps

Create a test file with intentional documentation gaps:

```python
# test_docs.py - file with documentation gaps for validation

def well_documented(param1: str, param2: int) -> bool:
    """Check if parameters are valid.

    Args:
        param1: The string to validate
        param2: The number to check

    Returns:
        True if valid, False otherwise
    """
    return bool(param1) and param2 > 0

def undocumented_function(data, options=None):
    # No docstring - should be flagged
    return process(data, options)

class DocumentedClass:
    """A well-documented class."""
    pass

class UndocumentedClass:
    # No docstring - should be flagged
    def method_without_docs(self):
        pass
```

**Expected detections in audit mode:**

- `undocumented_function` flagged as missing docs
- `UndocumentedClass` flagged as missing docs
- `method_without_docs` flagged as missing docs
- `well_documented` and `DocumentedClass` shown as documented

### Common Failure Modes and Fixes

| Failure Mode | Symptoms | Fix |
|--------------|----------|-----|
| Wrong format detected | JSDoc generated for Python file | Specify `format` parameter explicitly |
| Over-generation | Docs created for private/internal items | Set `scope=public` |
| Stale not detected | Old docs not flagged | Check `max_age_days` setting; verify git history available |
| Link check timeout | Report never completes | Set `check_links=false` for large projects |
| Coverage inflated | 100% but items missing | Check if only counting files, not functions |
| Generated docs wrong | Descriptions don't match behavior | Agent may need code context; check target_path |
| Encoding errors | Unicode issues in docs | Ensure files are UTF-8 encoded |
| Permission denied | Cannot write generated docs | Check file permissions on target directories |

### Pre-flight Checklist for Generate/Update Mode

Before running with `mode=generate` or `mode=update`:

```bash
# 1. Verify clean git state (can revert if needed)
git status --short
# Should be clean or only expected changes

# 2. Run audit first to see what will be generated
# Use mode=audit to preview changes

# 3. Verify target path is correct
ls -la {target_path}

# 4. Check existing doc format to ensure consistency
head -20 $(find . -name "*.py" -exec grep -l '"""' {} \; | head -1)

# 5. Create backup branch (optional but recommended)
git checkout -b backup/pre-docs-$(date +%Y%m%d) 2>/dev/null && git checkout -
```

### Quick Validation Script

Run these commands to validate agent readiness:

```bash
# 1. Verify project structure
ls -la README.md docs/ 2>/dev/null || echo "Standard doc locations"

# 2. Check for existing documentation format
# Python
grep -r '"""' --include="*.py" . 2>/dev/null | head -3

# JavaScript
grep -r '/\*\*' --include="*.js" . 2>/dev/null | head -3

# 3. Count documented vs undocumented (Python example)
echo "Files with docstrings:"
find . -name "*.py" -exec grep -l '"""' {} \; 2>/dev/null | wc -l

echo "Python files total:"
find . -name "*.py" ! -path "./venv/*" ! -path "./.venv/*" 2>/dev/null | wc -l

# 4. Check for README in key directories
find . -name "README.md" -not -path "./node_modules/*" -not -path "./.git/*"

# 5. Look for broken internal links in markdown
grep -r '\[.*\](.*\.md)' --include="*.md" . 2>/dev/null | head -10
```

---

*Maintain comprehensive, accurate documentation with automated coverage tracking.*
