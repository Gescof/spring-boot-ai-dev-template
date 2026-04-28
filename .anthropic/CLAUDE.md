# Spring Boot AI Dev Template — Claude Code Guide

## Project purpose

This is a starting point for new Spring Boot 3 + Java 21 + PostgreSQL services. Clone it, rename the coordinates, and generate your first features using the AI agents in `agents/`.

## Stack

- **Spring Boot 3** / Java 21
- **Spring Data JPA** + PostgreSQL 16
- **Jakarta Validation** + Spring Boot Actuator
- **Lombok** (constructor injection via `@RequiredArgsConstructor`)
- **JUnit 5** + Mockito + **Testcontainers** (real PostgreSQL in tests)
- **Docker Compose** for local dev; multi-stage Dockerfile for prod

## Running the project

```bash
# Full stack (app + PostgreSQL)
docker compose up --build

# Tests (Testcontainers starts PostgreSQL automatically)
./mvnw verify

# Local without Docker (set env vars first)
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/appdb \
SPRING_DATASOURCE_USERNAME=app \
SPRING_DATASOURCE_PASSWORD=app \
./mvnw spring-boot:run
```

Health check: `GET http://localhost:8080/actuator/health`

## Architecture rules

Follow these strictly when generating or modifying code:

- **Layer order**: `Controller → Service → Repository`. Never skip a layer.
- **No business logic in controllers.** No data access in controllers.
- **DTOs at the API boundary.** Never return JPA entities from controllers.
- **Constructor injection only.** Use `@RequiredArgsConstructor`; never `@Autowired` on fields.
- **Transactions on service methods.** `@Transactional` for writes, `@Transactional(readOnly = true)` for reads.
- **`@Getter`/`@Setter` on entities**, not `@Data` — Lombok's `@Data` generates `equals`/`hashCode` that breaks JPA identity.

## Package conventions

```
com.github.gescof.springbootaidevtemplate.
├── controller/
├── service/
├── repository/
├── model/
│   ├── entity/
│   ├── dto/
│   └── mapper/
└── exception/
```

When adding a new resource, create all layers. Never create a controller without a service.

## AI agents

The `agents/` directory contains structured prompts for common tasks. Use them directly:

```bash
# In Claude Code, read the agent and describe your task
cat agents/rest-api.md
# Then ask: "Using this agent, generate a REST API for Product management. A Product has..."
```

| Agent | File | Use for |
|---|---|---|
| **Orchestrator** | `agents/orchestrator.md` | Full feature: design → code → tests → review |
| REST API | `agents/rest-api.md` | New entity + CRUD endpoints (standalone) |
| Test Generator | `agents/test-generator.md` | Unit, controller, and integration tests |
| Code Review | `agents/code-review.md` | Review before committing |
| Microservice Design | `agents/microservice.md` | Architecture for a new service |

Start with the Orchestrator for new features — it delegates to the other agents in the correct order with quality gates between steps. Use individual agents only when you need one specific output in isolation.

See `docs/ai-usage.md` for detailed examples with each agent.

## Testing

- All integration tests extend `BaseIntegrationTest` at `src/test/java/.../BaseIntegrationTest.java`
- `BaseIntegrationTest` manages a real PostgreSQL container via Testcontainers
- Run all tests: `./mvnw verify`
- Do not create new Testcontainers configurations — always extend the base class

## Environment variables

| Variable | Default | Description |
|---|---|---|
| `SPRING_DATASOURCE_URL` | `jdbc:postgresql://postgres:5432/appdb` | JDBC URL |
| `SPRING_DATASOURCE_USERNAME` | `app` | DB username |
| `SPRING_DATASOURCE_PASSWORD` | `app` | DB password |
| `SERVER_PORT` | `8080` | HTTP port |

## Adapting for a new project

1. Update `pom.xml`: `groupId`, `artifactId`, `description`
2. Rename the Java package from `com.github.gescof.springbootaidevtemplate` to your own
3. Update the datasource defaults in `application.yml` if needed
4. Generate your first feature using `agents/orchestrator.md` (or `agents/rest-api.md` for a single resource)
