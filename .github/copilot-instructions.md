---
description: "Global policy for enterprise Spring Boot and Spring WebFlux Copilot customization framework."
applyTo: "**/*"
---

# Enterprise Copilot Framework Policy

## Architecture of the customization framework
- Agents orchestrate only.
- Skills contain implementation knowledge.
- Instructions contain shared standards.

## Primary orchestration model
- Use `.github/agents/supervisor.agent.md` as the single entry point.
- Route to specialist agents based on request intent.
- Aggregate outputs and return one final result.

## Existing-project first rule
For existing applications, always run architecture review before generation:
1. Structure analysis
2. Architecture style detection
3. Convention detection
4. Framework detection
5. Security pattern detection
6. Testing pattern detection

Use: `.github/skills/review-architecture`.

## Supported architecture styles
- Layered Architecture
- Hexagonal Architecture
- Event-Driven Architecture
- Domain-Driven Design

Generation must adapt to detected style and local conventions.

## Supported application types
- Traditional enterprise applications
- AI-agent-based applications

## Technology expectations
- Spring Boot and Spring WebFlux
- Reactive-first design in request paths
- Security, observability, and testing are mandatory workflow stages

## Shared standards source of truth
- `.github/instructions/coding-standards.md` - Code organization and style conventions
- `.github/instructions/architecture.md` - Architectural patterns and boundaries
- `.github/instructions/webflux-api.instructions.md` - WebFlux API implementation standards
- `.github/instructions/security-hardening.instructions.md` - Spring Security and endpoint hardening
- `.github/instructions/logging-observability.instructions.md` - Logging, tracing, and metrics standards
- `.github/instructions/testing-testcontainers.instructions.md` - Testing patterns and Testcontainers setup
- `.github/instructions/exception-handling.instructions.md` - Global exception handling and error responses

## Implementation source of truth
- Skills in `.github/skills/*/SKILL.md`

## Prompt entry points
- `.github/prompts/add-feature.md`
- `.github/prompts/create-api.md`
- `.github/prompts/create-ai-agent.md`
- `.github/prompts/create-event-handler.md`
- `.github/prompts/fix-bug.md`
- `.github/prompts/refactor-feature.md`
- `.github/prompts/review-architecture.md`
