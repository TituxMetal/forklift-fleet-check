# Feature Shape Template

## Overview

A Feature Shape is a mid-level planning document that bridges the gap between the high-level MVP
definition and the concrete implementation steps. It describes **what** to build and **why**,
without prescribing **how** (no file names, no code).

Feature Shapes are created just-in-time — only 2-3 features ahead of current work.

---

## Template

```markdown
# Feature Shape: [Feature Name]

## Problem

What user problem does this feature solve? Why does it need to exist? Keep it short — 2-3 sentences
max.

## Solution (Broad Strokes)

Describe the solution at a high level:

- What can the user do?
- What are the main UI elements/screens?
- What data is involved?

Do NOT specify:

- File names or paths
- Function signatures
- Specific implementation details

## User Flow

Step-by-step description of how the user interacts with this feature:

1. User does X
2. System shows Y
3. User does Z
4. etc.

## Dependencies

**Requires:**

- What must exist before this feature can be built?
- Other features, infrastructure, data

**Enables:**

- What features depend on this one?
- Why is this feature needed for the next steps?

## What Must Exist (Backend)

High-level list of backend components needed:

- Entities / data models
- Use cases / operations
- API endpoints
- Validations

Do NOT specify file names — just describe what's needed functionally.

## What Must Exist (Frontend)

High-level list of frontend components needed:

- Pages / routes
- Components (list, form, modal, etc.)
- State management needs
- User interactions

Do NOT specify file names — just describe what's needed functionally.

## Open Questions

Things that need clarification before or during implementation:

1. Question 1?
2. Question 2?

These can be resolved during the implementation session.

## Out of Scope

What this feature explicitly does NOT include. Helps prevent scope creep during implementation.

## Risks / Gotchas

Potential issues, edge cases, or tricky parts to watch out for:

- Technical risks
- UX concerns
- Data integrity issues
- Things that might be forgotten
```

---

## Guidelines

### When to Create a Feature Shape

- Before starting implementation of a feature
- Only for the next 2-3 features (just-in-time planning)
- After the codebase context is available (clone the project first)

### Who Creates It

- You + Claude Code together
- Claude Code reads the existing code to understand conventions
- You validate and adjust based on your knowledge

### How Detailed Should It Be

**Too vague:**

> "Admin can manage equipment"

**Too detailed:**

> "Create file `src/domain/entities/equipment.entity.ts` with class Equipment..."

**Just right:**

> "Admin can create, edit, and deactivate equipment. Each equipment has a name, internal code, CACES
> category, ownership type, and status. The status is calculated from inspections, not manually
> editable."

### Living Document

Feature Shapes can be updated during implementation if:

- New open questions are discovered
- Scope needs adjustment
- Dependencies change

But avoid major rewrites — if the shape was wrong, learn from it for the next one.
