---
name: fix-bug
description: Diagnose and fix defects with root-cause-first workflow and regression-safe validation.
---

# Skill: Fix Bug

## Workflow
1. Reproduce issue and capture expected vs actual behavior.
2. Trace flow across controller, service, repository, integration, and handler layers.
3. Identify root cause and apply minimal-impact fix.
4. Add regression tests.

## Best practices
- Root cause before patching.
- Avoid broad refactors during incident fixes.
- Preserve security and observability controls.

## Validation criteria
- Bug scenario is reproducible before and resolved after fix.
- Regression test exists.
- No new warnings in relevant layers.
