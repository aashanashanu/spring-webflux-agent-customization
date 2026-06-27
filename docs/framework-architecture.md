# Enterprise GitHub Copilot Customization Framework

## Purpose
Reusable framework for Spring Boot and Spring WebFlux teams that separates:
- Agent orchestration responsibilities
- Skill implementation knowledge
- Shared engineering standards

## Responsibility model
- Agents: intent analysis, routing, orchestration, aggregation.
- Skills: templates, implementation guidance, best practices, examples, validation.
- Instructions: cross-cutting standards applied consistently.
- Prompts: reusable entry points that leverage Orchestrator orchestration.

## Component map
```mermaid
flowchart TD
    U[Developer Request] --> S[Orchestrator Agent]
    S --> A[Architecture Agent]
    A --> SKR[Skill: review-architecture]

    S --> API[API Agent]
    S --> PERS[Persistence Agent]
    S --> INT[Integration Agent]
    S --> SEC[Security Agent]
    S --> OBS[Observability Agent]
    S --> TST[Testing Agent]
    S --> AIA[AI-Agent Agent]

    API --> SK1[Skills: create-api/create-service]
    PERS --> SK2[Skills: create-repository]
    INT --> SK3[Skills: webclient/kafka/sqs/eventbridge/mcp]
    SEC --> SK4[Skill: secure-endpoint]
    TST --> SK5[Skills: write-unit-tests/write-integration-tests]
    AIA --> SK6[Skills: create-ai-agent/create-mcp-tool]

    SK1 --> OUT[Aggregated Output]
    SK2 --> OUT
    SK3 --> OUT
    SK4 --> OUT
    SK5 --> OUT
    SK6 --> OUT
```

## Workflow patterns

### Greenfield workflow
```mermaid
flowchart LR
    R[Request] --> S[Orchestrator]
    S --> A[Architecture]
    A --> API[API]
    API --> P[Persistence]
    P --> I[Integration]
    I --> SEC[Security]
    SEC --> O[Observability]
    O --> T[Testing]
    T --> D[Delivery Summary]
```

### Existing project workflow
```mermaid
flowchart LR
    R[Request] --> S[Orchestrator]
    S --> A[Architecture + review-architecture]
    A --> C[Convention Detection]
    C --> X[Style-adapted Generation]
    X --> T[Regression Testing]
    T --> D[Delivery Summary]
```

### AI-agent workflow
```mermaid
flowchart LR
    R[Request] --> S[Orchestrator]
    S --> A[Architecture]
    A --> AI[AI-Agent]
    AI --> I[Integration]
    I --> SEC[Security]
    SEC --> O[Observability]
    O --> T[Testing]
    T --> D[Delivery Summary]
```

## Architectural style awareness
Supported styles:
- Layered Architecture
- Hexagonal Architecture
- Event-Driven Architecture
- Domain-Driven Design

Generation adapts by style and detected conventions.

## Existing application adaptation checklist
Before generation:
1. Analyze structure and module boundaries.
2. Detect architecture style.
3. Detect coding and naming conventions.
4. Detect frameworks and integrations.
5. Detect security patterns.
6. Detect testing patterns.

## Integration support matrix
- REST APIs
- WebClient
- Kafka
- SQS
- EventBridge
- OpenSearch/Elasticsearch
- AWS SDK
- Bedrock
- MCP servers/tools

## Observability coverage
- Micrometer metrics
- Prometheus scrape compatibility
- OpenTelemetry tracing
- Correlation ID propagation
- Structured logging
- Health checks and operational readiness

## Testing coverage
- Unit tests
- Integration tests
- Testcontainers
- StepVerifier
- WebTestClient
- Contract testing
- Performance testing

## Example orchestration request
```text
Use Orchestrator to add a new payments API in an existing WebFlux service.
Run architecture review first, adapt to existing style, implement endpoint + persistence + security,
and provide unit/integration tests with summary report.
```
