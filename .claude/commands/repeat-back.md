# Repeat Back Command

**Version:** 1.0.0
**Last Updated:** 2025-01-20

Repeat back the user's query with enhanced clarity and detail to confirm mutual understanding before proceeding. Use `$ARGUMENTS` as the query to analyze (or reference the most recent user request if empty).

## Usage

| Command | Action |
|---------|--------|
| `/repeat-back` | Clarify the most recent user request |
| `/repeat-back <query>` | Clarify the specified query |

## Instructions

When the user runs `/repeat-back`, perform the following steps:

### Step 1: Identify the Query

**If `$ARGUMENTS` provided:** Use it as the query to analyze.

**If no arguments:** Use the most recent substantive user request from the conversation.

### Step 2: Analyze the Query

Break down the query into its components:

1. **Core Intent** - What is the user fundamentally trying to accomplish?
2. **Explicit Requirements** - What did the user specifically ask for?
3. **Implicit Requirements** - What unstated needs can be reasonably inferred?
4. **Scope** - What are the boundaries of the request?
5. **Constraints** - Any limitations, preferences, or conditions mentioned?
6. **Ambiguities** - Any unclear aspects that need clarification?

### Step 3: Formulate the Repeat-Back

Present your understanding using this structure:

```
## My Understanding

**Core Goal:** [One sentence summarizing the fundamental objective]

**What You're Asking For:**
- [Explicit requirement 1]
- [Explicit requirement 2]
- ...

**What I'm Inferring:**
- [Implicit requirement or assumption 1]
- [Implicit requirement or assumption 2]
- ...

**Scope:**
- Includes: [What's in scope]
- Excludes: [What's out of scope, if relevant]

**Constraints/Preferences:**
- [Any mentioned constraints]

---

**Questions for Clarification:** (if any ambiguities exist)
1. [Question about unclear aspect]
2. [Question about unclear aspect]

---

Is this understanding correct? Please confirm or correct any misunderstandings before I proceed.
```

### Step 4: Wait for Confirmation

Do not proceed with the task until the user confirms or corrects your understanding.

**If confirmed:** Proceed with the task.

**If corrected:** Update your understanding and repeat back the corrected version.

---

## Guidelines

### Be Specific, Not Generic

**Bad:** "You want me to fix something in the code."

**Good:** "You want me to fix the authentication bug where users are logged out after 5 minutes, specifically in the session management module."

### Surface Assumptions

Make implicit assumptions explicit so the user can validate them:

**Bad:** "I'll update the function."

**Good:** "I'll update the `calculateTotal()` function. I'm assuming you want to preserve backward compatibility with existing callers."

### Identify Scope Boundaries

Help the user understand what you will and won't do:

**Example:** "This includes updating the API endpoint and its tests. This does not include updating the documentation or client-side code unless you'd like me to."

### Ask Targeted Questions

If aspects are unclear, ask specific questions rather than vague ones:

**Bad:** "Can you clarify what you mean?"

**Good:** "Should the new validation apply to existing records, or only to new records created after deployment?"

---

## When to Use

- Complex multi-step tasks
- Ambiguous or open-ended requests
- Tasks with significant consequences (refactoring, deletions, deployments)
- When you're uncertain about requirements
- Before starting time-intensive work

---

## Examples

### Example 1: Simple Clarification

**User:** "Add dark mode"

**Repeat-Back:**

```
## My Understanding

**Core Goal:** Implement a dark mode theme option for the application.

**What You're Asking For:**
- Add the ability for users to switch between light and dark themes

**What I'm Inferring:**
- This should include a toggle mechanism (likely in settings or header)
- Dark mode styles need to be created for all existing components
- User preference should persist across sessions

**Scope:**
- Includes: Theme toggle, dark color palette, persistent preference
- Excludes: Additional themes beyond light/dark (unless requested)

**Questions for Clarification:**
1. Should the theme respect the user's OS-level preference by default?
2. Where should the toggle be located (settings page, header, both)?

Is this understanding correct?
```

### Example 2: Technical Task

**User:** "Optimize the database queries, they're slow"

**Repeat-Back:**

```
## My Understanding

**Core Goal:** Improve database query performance to reduce response times.

**What You're Asking For:**
- Identify and optimize slow database queries

**What I'm Inferring:**
- I should first profile/analyze current query performance
- Focus on the most impactful optimizations (slowest or most frequent queries)
- Preserve existing functionality while improving performance

**Scope:**
- Includes: Query analysis, index recommendations, query rewrites
- Excludes: Schema redesign, database engine changes (unless critical)

**Constraints:**
- Changes should not break existing functionality

**Questions for Clarification:**
1. Are there specific endpoints or features where slowness is most noticeable?
2. Do you have query logs or metrics showing which queries are problematic?
3. Are there any constraints on adding new indexes (storage, write performance)?

Is this understanding correct?
```

---

## Related

- [AskUserQuestion tool](../GLOSSARY.md) - For asking specific clarifying questions
- `/dev-cycle` - For executing confirmed development tasks

---

## Version History

### 1.0.0 (2025-01-20)

- Initial release
- Structured repeat-back format with core goal, requirements, scope, and clarifications
- Guidelines for specificity and assumption surfacing
- Examples for common scenarios
