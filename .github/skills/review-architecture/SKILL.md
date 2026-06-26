---
name: review-architecture
description: Analyze an existing project before code generation to detect architecture style, conventions, and platform patterns.
---

# Skill: Review Architecture

## Mandatory use
This skill must run before generating code for existing applications.

## Analysis goals
1. Detect architecture style:
   - Layered Architecture
   - Hexagonal Architecture
   - Event-Driven Architecture
   - Domain-Driven Design
2. Detect existing coding conventions:
   - Packaging strategy
   - Naming patterns
   - DTO/entity boundaries
   - Error handling model
3. Detect frameworks and runtime patterns:
   - Spring modules, persistence stack, messaging stack
   - Build tool and plugin conventions
4. Detect security approach:
   - OAuth2/JWT, custom filters, route authorization model
5. Detect testing approach:
   - Unit framework, integration style, Testcontainers, contract/performance tests

## Output format
```markdown
Architecture detection summary:
- Primary style: <style>
- Secondary style: <style or none>

Convention summary:
- Package layout:
- Naming:
- DTO/entity boundaries:
- Error handling pattern:

Platform summary:
- Persistence:
- Security:
- Testing:

Generation guidance:
- Required constraints for downstream generation
- Do-not-break contracts
```

## Best practices
- Prefer evidence from code structure over assumptions.
- Flag uncertainty explicitly.
- Recommend style-compatible generation paths.

## Validation criteria
- Architecture style identified with evidence.
- Security/testing conventions captured.
- Actionable constraints produced for downstream skills.
