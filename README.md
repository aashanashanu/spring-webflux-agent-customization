# Enterprise Copilot Customization Framework

This repository is refactored into a reusable, enterprise-grade GitHub Copilot customization framework for Spring Boot and Spring WebFlux applications.

## Design goals

- Greenfield project generation support.
- Existing application enhancement support.
- Support for both traditional enterprise apps and AI-agent-based apps.
- Strong separation of orchestration, implementation knowledge, and shared standards.

## Framework structure

```text
.github/
  agents/
    orchestrator.agent.md
    architecture.agent.md
    api.agent.md
    persistence.agent.md
    integration.agent.md
    security.agent.md
    observability.agent.md
    testing.agent.md
    ai-agent.agent.md
  skills/
    create-api/
    create-service/
    create-repository/
    create-webclient/
    create-event-handler/
    secure-endpoint/
    create-ai-agent/
    create-mcp-tool/
    create-kafka-consumer/
    create-kafka-producer/
    create-sqs-consumer/
    create-eventbridge-integration/
    write-unit-tests/
    write-integration-tests/
    refactor-feature/
    fix-bug/
    review-architecture/
  instructions/
    coding-standards.md
    webflux.md
    testing.md
    security.md
    logging.md
    architecture.md
    observability.md
  prompts/
    add-feature.md
    create-api.md
    create-ai-agent.md
    create-event-handler.md
    fix-bug.md
    refactor-feature.md
    review-architecture.md

docs/
  framework-architecture.md
  refactor-summary.md
```

## Responsibility boundaries

### Agents (orchestration only)

- Understand user intent.
- Route tasks to specialist agents.
- Sequence workflow stages.
- Pass context and aggregate output.
- Avoid implementation templates and coding standards.

### Skills (implementation knowledge)

- Hold templates and examples.
- Contain domain best practices.
- Provide implementation guidance.
- Define validation criteria.

### Instructions (shared standards)

- Define organization-wide coding and runtime rules.
- Provide cross-cutting standards for architecture, security, testing, logging, and observability.

## Orchestrator workflow

Orchestrator is the primary entry point for developers.

### New API request

Architecture -> API -> Persistence -> Security -> Observability -> Testing

### Bug-fix request

Architecture Review -> Relevant Specialist -> Testing

### New AI-agent request

Architecture -> AI-Agent -> Integration -> Security -> Observability -> Testing

## Existing application workflow (mandatory)

Before generating code:

1. Analyze project structure.
2. Detect architecture style.
3. Detect coding conventions.
4. Detect frameworks.
5. Detect security patterns.
6. Detect testing patterns.

This workflow is implemented by `skills/review-architecture` and must run first for existing applications.

## Supported architecture styles

- Layered Architecture
- Hexagonal Architecture
- Event-Driven Architecture
- Domain-Driven Design

Generation adapts to detected or selected style.

## Integration support

- REST APIs
- WebClient
- Kafka
- SQS
- EventBridge
- OpenSearch / Elasticsearch
- AWS SDK
- Bedrock
- MCP servers/tools

## Testing support

- Unit tests
- Integration tests
- Testcontainers
- StepVerifier
- WebTestClient
- Contract testing
- Performance testing

## Additional docs

- See `docs/framework-architecture.md` for diagrams and workflows.
- See `docs/refactor-summary.md` for final validation and change report.
