---
name: refactor-feature
description: Refactor existing features safely while preserving behavior and project conventions.
---

# Skill: Refactor Feature

## Workflow
1. Capture current behavior and dependencies.
2. Define refactor scope and non-goals.
3. Apply incremental changes aligned with architecture style.
4. Validate no behavior regressions.

## Best practices
- Refactor in small commits/steps.
- Preserve public contracts unless explicitly changing version.
- Add/adjust tests before risky refactors.

## Example
- Extract duplicated mapping logic into mapper component.

## Validation criteria
- Existing tests pass.
- New tests cover refactored paths.
- Public API and error contracts unchanged unless requested.
