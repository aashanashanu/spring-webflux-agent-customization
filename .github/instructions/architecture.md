# Architecture Standards

## Supported styles
- Layered Architecture.
- Hexagonal Architecture.
- Event-Driven Architecture.
- Domain-Driven Design.

## Selection and adaptation
- For greenfield work, choose style by domain complexity and integration needs.
- For existing applications, detect and align to existing style before code generation.
- Keep style boundaries explicit (inbound adapters, application services, domain, outbound adapters where relevant).

## Modifiability
- Preserve existing package/module boundaries unless explicitly refactoring.
- Keep integration boundaries and contracts stable.
