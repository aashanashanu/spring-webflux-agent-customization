---
name: webflux-feature-workflow
description: "Use when building a new Spring Boot WebFlux feature module end-to-end — covers database selection (SQL, Couchbase, or MongoDB), controller, service, repository, DTO, logging, Spring Security route protection, global exception handling, and Testcontainers integration tests."
---

# WebFlux Feature Workflow (Modular)

This workflow skill orchestrates the reusable skill library for end-to-end feature delivery.

## Flow selection
- Greenfield: architecture -> create-api -> create-service -> create-repository -> secure-endpoint -> write-unit-tests -> write-integration-tests
- Existing application: review-architecture -> architecture-aware adaptation -> feature skills -> regression testing
- Event-driven integration: create-event-handler + kafka/sqs/eventbridge skills + integration tests
- AI-agent integration: create-ai-agent + create-mcp-tool + integration + security + testing

## Required pre-checks
1. Confirm target mode (greenfield or existing).
2. Detect architecture style (Layered, Hexagonal, Event-Driven, DDD).
3. Detect security and testing conventions for existing projects.

## Skill routing map
- API and service changes: `skills/create-api`, `skills/create-service`
- Persistence changes: `skills/create-repository`
- External APIs: `skills/create-webclient`
- Eventing: `skills/create-event-handler`, `skills/create-kafka-consumer`, `skills/create-kafka-producer`, `skills/create-sqs-consumer`, `skills/create-eventbridge-integration`
- Security: `skills/secure-endpoint`
- AI-agent capabilities: `skills/create-ai-agent`, `skills/create-mcp-tool`
- Bug/refactor: `skills/fix-bug`, `skills/refactor-feature`
- Testing: `skills/write-unit-tests`, `skills/write-integration-tests`

## Output requirements
- Generated/modified artifact list.
- Architecture compatibility summary.
- Validation summary with test scope and remaining risks.
