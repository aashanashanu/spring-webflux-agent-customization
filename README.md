# Spring Boot WebFlux — GitHub Copilot Agent Customization

This repository contains a complete **GitHub Copilot agent customization setup** for Spring Boot WebFlux services.  
It provides prompts, instructions, skills, agents, and guardrail hooks that steer Copilot to produce consistent, production-ready reactive code — covering API design, persistence, security, logging, exception handling, and testing.

---

## Table of Contents

1. [What Is Included](#what-is-included)  
2. [Quick Start — Generate a New Spring Boot Application](#quick-start--generate-a-new-spring-boot-application)  
3. [Adding Features to an Existing Application](#adding-features-to-an-existing-application)  
4. [Using the Prompts](#using-the-prompts)  
5. [Using the Skills](#using-the-skills)  
6. [Using the Agents](#using-the-agents)  
7. [Using the Instructions](#using-the-instructions)  
8. [Guardrail Hooks](#guardrail-hooks)  
9. [MCP Context Contract](#mcp-context-contract)  
10. [How to Customize](#how-to-customize)  
11. [Configuration Reference](#configuration-reference)  

---

## What Is Included

```
.github/
├── copilot-instructions.md          # Global policy applied to every Copilot interaction
├── prompts/
│   ├── add-new-api.prompt.md        # Guided prompt: add a versioned REST API endpoint
│   └── bug-fix.prompt.md            # Guided prompt: diagnose and fix a bug
├── skills/
│   └── webflux-feature-workflow/
│       └── SKILL.md                 # End-to-end workflow for building a new feature module
├── agents/
│   ├── api-designer.agent.md        # Controller, DTO, route design
│   ├── error-handling.agent.md      # Exception handler, error model, domain exceptions
│   ├── persistence-couchbase-sql.agent.md  # Repositories, entities, DB config (SQL/Couchbase/MongoDB)
│   ├── security-hardening.agent.md  # Spring Security WebFlux, JWT, HTTP headers
│   └── testcontainers-test.agent.md # Integration & API tests with Testcontainers
├── instructions/
│   ├── exception-handling.instructions.md  # Rules for exception/error files
│   ├── logging-observability.instructions.md  # Rules for logback + logging filters
│   ├── security-hardening.instructions.md  # Rules for security config files
│   ├── testing-testcontainers.instructions.md  # Rules for test files
│   └── webflux-api.instructions.md  # Rules for controller/service/DTO files
├── hooks/
│   ├── pretool-guardrails.json      # Pre-generation guardrails (blocking unsafe code)
│   └── posttool-quality-gates.json  # Post-generation quality reminders
└── mcp/
    └── context-contract.json        # MCP context schema: project baseline + policy
```

---

## Quick Start — Generate a New Spring Boot Application

### 1. Open the workspace in VS Code

Clone or copy this repo, then open the folder in VS Code with GitHub Copilot enabled.

### 2. Start a new Chat (Agent mode)

Open the Copilot Chat panel (`Ctrl+Alt+I` / `Cmd+Alt+I`) and switch to **Agent** mode.

### 3. Trigger the end-to-end feature workflow

Type in Copilot Chat:

```
Use the webflux-feature-workflow skill to scaffold a new Spring Boot WebFlux service for managing [your resource, e.g. orders].
```

Copilot will read `.github/skills/webflux-feature-workflow/SKILL.md` and walk you through:

| Step | What is generated |
|------|-------------------|
| 0    | Gradle `build.gradle`, Spring Actuator config, Swagger/OpenAPI config |
| 1    | Database choice (SQL → R2DBC or NoSQL → Couchbase / MongoDB) |
| 2    | Controller, Service, Repository, Entity/Document, DTOs, Mapper |
| 3    | `RequestResponseLoggingFilter`, `logback-spring.xml` rolling policy |
| 4    | `SecurityWebFilterChain` with route protection and HTTP security headers |
| 5    | `GlobalExceptionHandler`, `ApiErrorResponse`, domain exception classes |
| 6    | Testcontainers integration tests for all HTTP scenarios |
| 7    | Final checklist verification |

### 4. Answer the prompts

Copilot will ask:
- Resource name (e.g. `Order`)
- Database type and engine
- Which HTTP operations to expose
- Field definitions for request/response DTOs
- Access control requirements

---

## Adding Features to an Existing Application

### Add a new API endpoint

Use the `add-new-api` prompt. In Copilot Chat:

```
/add-new-api
```

or reference it explicitly:

```
Use the prompt .github/prompts/add-new-api.prompt.md to add a new API for [resource].
```

The prompt enforces:
- API versioning at `/api/v{n}/...`
- Separate request/response DTOs with validation annotations
- `@Operation` / `@ApiResponse` OpenAPI annotations
- Inline comments at required locations
- `SecurityWebFilterChain` route registration
- Reactive return types throughout

### Fix a bug

Use the `bug-fix` prompt:

```
/bug-fix
```

or:

```
Use the prompt .github/prompts/bug-fix.prompt.md. The bug is: [describe symptom].
```

The prompt guides Copilot through:
1. Symptom gathering
2. Root-cause analysis using the reactive call chain
3. Minimal-impact fix with required inline comments
4. Regression check and test coverage
5. Security review

### Add persistence only

Delegate directly to the persistence agent:

```
Use the persistence-couchbase-sql agent to add a repository and entity for [resource] using PostgreSQL.
```

### Add tests only

```
Use the testcontainers-test agent to generate integration tests for the [resource] controller.
```

---

## Using the Prompts

Prompt files are in `.github/prompts/`. They can be invoked in two ways:

**Slash command** (if configured in VS Code):

```
/add-new-api
/bug-fix
```

**Explicit reference in chat**:

```
Use the prompt in .github/prompts/add-new-api.prompt.md
```

| Prompt | Purpose |
|--------|---------|
| `add-new-api.prompt.md` | Add a versioned REST endpoint with full DTO, validation, OpenAPI docs, security, and tests |
| `bug-fix.prompt.md` | Systematic bug diagnosis and minimal-impact fix with regression tests |

---

## Using the Skills

Skills encode **end-to-end multi-step workflows**. They are longer than instructions and guide Copilot through a complete task across multiple files.

**How to invoke**:

```
Use the webflux-feature-workflow skill to build [feature].
```

| Skill | File | Purpose |
|-------|------|---------|
| `webflux-feature-workflow` | `.github/skills/webflux-feature-workflow/SKILL.md` | Complete feature scaffold: baseline setup, database, API layers, security, exception handling, tests |

### Customizing a skill

1. Open `.github/skills/webflux-feature-workflow/SKILL.md`
2. Edit any step — add new steps, adjust code templates, or change default values
3. The change takes effect immediately on the next Copilot Chat invocation

---

## Using the Agents

Agents are **specialized sub-workers** focused on one layer. Copilot invokes them automatically when context matches, or you can invoke them explicitly.

**Explicit invocation**:

```
Use the api-designer agent to create a controller for [resource].
Use the error-handling agent to add a new custom exception for [scenario].
Use the security-hardening agent to protect the /api/v1/payments route.
```

| Agent | File | Responsibility |
|-------|------|---------------|
| `api-designer` | `agents/api-designer.agent.md` | Controllers, DTOs, routes, OpenAPI annotations |
| `error-handling` | `agents/error-handling.agent.md` | `GlobalExceptionHandler`, domain exceptions, `ApiErrorResponse` |
| `persistence-couchbase-sql` | `agents/persistence-couchbase-sql.agent.md` | Repositories, entities/documents, DB config (SQL/Couchbase/MongoDB) |
| `security-hardening` | `agents/security-hardening.agent.md` | `SecurityWebFilterChain`, JWT, HTTP headers, route protection |
| `testcontainers-test` | `agents/testcontainers-test.agent.md` | Integration tests, `WebTestClient`, error contract assertions |

### Adding a new agent

1. Create `.github/agents/<name>.agent.md`
2. Add a YAML frontmatter block:

```yaml
---
name: my-agent
description: "One-sentence description of when to invoke this agent."
tools:
  - read_file
  - file_search
  - grep_search
  - create_file
  - replace_string_in_file
---
```

3. Write the agent's role, constraints, code templates, and checklist below the frontmatter

---

## Using the Instructions

Instructions are **file-pattern scoped rules** automatically applied by Copilot when it opens a matching file. You do not need to invoke them manually.

| File | Applied to | Enforces |
|------|-----------|---------|
| `webflux-api.instructions.md` | `*Controller.java`, `*Service.java`, `dto/**` | Controller rules, DTO design, reactive patterns, versioned routes |
| `exception-handling.instructions.md` | `*Exception*.java`, `*ExceptionHandler*.java`, `exception/**` | Exception hierarchy, error payload shape, status code mapping |
| `logging-observability.instructions.md` | `logback*.xml`, `*LoggingFilter.java`, `*WebFilter.java` | Log format, rotation policy, sensitive field masking |
| `security-hardening.instructions.md` | `*SecurityConfig*.java`, `application*.yml` | Deny-by-default, permitted routes, HTTP headers, CSRF comment |
| `testing-testcontainers.instructions.md` | `src/test/**`, `*IT.java`, `*Test.java` | Testcontainers setup, mandatory test coverage scenarios, `WebTestClient` patterns |

### Adding a new instruction file

1. Create `.github/instructions/<name>.instructions.md`
2. Add a YAML frontmatter block:

```yaml
---
description: "Short description of when this applies."
applyTo: "**/pattern/**/*.java,**/OtherPattern.java"
---
```

3. Write the rules below the frontmatter using Markdown
4. The `applyTo` glob pattern determines which files auto-trigger these rules

### Editing an existing instruction

Open the relevant file in `.github/instructions/` and edit the rule text. Changes take effect on the next Copilot Chat session.

---

## Guardrail Hooks

Hooks in `.github/hooks/` run automatically before and after tool use to enforce safety rules.

### Pre-tool guardrails (`pretool-guardrails.json`)

| Hook name | Triggers when | Action |
|-----------|--------------|--------|
| `enforce-database-choice` | Creating Repository/Entity/Document files | Reminds to confirm database choice first |
| `reject-unsupported-nosql` | Code referencing Cassandra or Redis repositories | **Blocks** with an error message |
| `no-blocking-in-reactive-path` | `.block()` / `Thread.sleep` in service/controller files | **Blocks** with a warning |
| `exception-contract-required` | Creating a Controller file | Reminds to ensure `GlobalExceptionHandler` exists |
| `no-secrets-in-config` | Hardcoded secrets in `application.yml` | **Blocks** with a security error |

### Post-tool quality gates (`posttool-quality-gates.json`)

| Hook name | Triggers when | Reminder |
|-----------|--------------|---------|
| `remind-run-tests` | Any `.java` file created | Run `./gradlew test integrationTest` |
| `check-exception-handler-coverage` | New exception class created | Map it in `GlobalExceptionHandler` |
| `security-log-scan` | Logging filter modified | Verify sensitive fields are masked |
| `verify-reactive-return-types` | Service or Controller modified | Check all returns are `Mono`/`Flux` |
| `check-dto-validation-annotations` | Request DTO created | Verify `@NotNull`/`@Valid` annotations |
| `verify-security-route-updated` | New controller created | Update `SecurityWebFilterChain` |
| `check-test-coverage` | Any `src/main/java` file created | Verify corresponding test file exists |

### Adding a custom guardrail

Add an entry to `pretool-guardrails.json` or `posttool-quality-gates.json`:

```json
{
  "event": "PreToolUse",
  "name": "my-guardrail",
  "description": "What this checks.",
  "matcher": {
    "tool": "create_file",
    "pathContains": ["MyPattern"]
  },
  "run": "echo '[GUARDRAIL] My check message.' && exit 0"
}
```

Use `exit 1` to block the action; `exit 0` to allow and print a reminder only.

---

## MCP Context Contract

`.github/mcp/context-contract.json` is a machine-readable schema that defines all project-level defaults and enforcements. It is consulted by MCP-compatible tooling and agents.

Key sections:

| Section | What it controls |
|---------|-----------------|
| `context.project` | Build tool (`gradle`), Java version, Spring Boot version |
| `context.platform` | Actuator endpoints, OpenAPI paths |
| `context.database` | Allowed DB types and engines |
| `context.logging` | Log rotation config, sensitive field list |
| `context.security` | Stateless JWT, deny-by-default |
| `context.exception_handling` | Error payload fields, exception-to-status mapping table |
| `context.testing` | Testcontainers images, mandatory test cases |
| `policy` | Boolean enforcement flags (require Gradle, Actuator, Swagger, etc.) |

### Changing a default

Edit the relevant key in `context-contract.json`. For example, to change the exposed Actuator endpoints:

```json
"actuator": {
  "enabled": true,
  "exposed_endpoints": ["health", "info", "metrics", "env"],
  "health_show_details": "always"
}
```

---

## Configuration Reference

### Mandatory project baseline (enforced by policy)

| Requirement | What to use | Config location |
|-------------|-------------|-----------------|
| Build tool | Gradle — `build.gradle` / `build.gradle.kts` | `copilot-instructions.md` §1 |
| Spring Actuator | `spring-boot-starter-actuator` | `copilot-instructions.md` §1.1 |
| Swagger / OpenAPI | `springdoc-openapi-starter-webflux-ui` | `copilot-instructions.md` §1.1 |
| Actuator endpoints | `health`, `info`, `metrics` exposed by default | `application.yml` |
| OpenAPI docs | `/v3/api-docs` | `application.yml` / springdoc config |
| Swagger UI | `/swagger-ui.html` | `application.yml` / springdoc config |

### Supported database stacks

| Type | Engine | Spring library |
|------|--------|---------------|
| SQL | PostgreSQL | `spring-boot-starter-data-r2dbc` + `r2dbc-postgresql` |
| SQL | MySQL | `spring-boot-starter-data-r2dbc` + `r2dbc-mysql` |
| SQL | MariaDB | `spring-boot-starter-data-r2dbc` + `r2dbc-mariadb` |
| NoSQL | Couchbase | `spring-boot-starter-data-couchbase-reactive` |
| NoSQL | MongoDB | `spring-boot-starter-data-mongodb-reactive` |

### Key `application.yml` defaults

```yaml
# Spring Actuator — minimal endpoint exposure
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
  endpoint:
    health:
      show-details: when_authorized

# SpringDoc OpenAPI / Swagger UI
springdoc:
  api-docs:
    path: /v3/api-docs
  swagger-ui:
    path: /swagger-ui.html
```

---

## Changing Global Policy

The single source of truth for global rules is `.github/copilot-instructions.md`.  
Edit any section there to change defaults that apply to **every** Copilot session in this workspace:

- §1 Architecture & Coding Standards — add or remove coding rules
- §1.1 Platform Baseline — change which dependencies are mandatory
- §2 Database Decision Rule — add or remove supported engines
- §3 REST Controller Conventions — adjust response type rules
- §6 Spring Security Hardening — change header or auth requirements
- §7 Global Exception Handling — adjust error payload fields or status mappings
- §8 Testcontainers — change container images or mandatory test scenarios
