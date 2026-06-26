# Coding Standards

## Core rules

- Prefer constructor injection.
- Use records for immutable DTOs where appropriate.
- Favor immutability for request/response and value objects.
- Keep methods small and purpose-specific.
- Use explicit names for classes, methods, and variables.

## Naming conventions

- Classes: PascalCase.
- Methods/fields: camelCase.
- Constants: UPPER_SNAKE_CASE.
- Test methods: `should_<outcome>_when_<condition>`.

## Design rules

- Thin controllers, rich service/application logic.
- No magic numbers; use named constants.
- Avoid duplicated logic; extract shared utilities.
- Prefer composition over inheritance unless inheritance is required.
