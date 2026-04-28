# Spring Boot AI Dev Template

A production-ready Spring Boot 3 template designed for AI-assisted development. Use it as a starting point for new projects and leverage the included AI agent prompts to generate code, tests, and architecture with Claude, GitHub Copilot, or Cursor.

## Stack

| Layer | Technology |
|---|---|
| Framework | Spring Boot 3 + Java 21 |
| Persistence | Spring Data JPA + PostgreSQL 16 |
| Validation | Jakarta Validation |
| Observability | Spring Boot Actuator |
| Testing | JUnit 5 + Mockito + Testcontainers |
| Build | Maven + Maven Wrapper |
| Containers | Docker + Docker Compose |
| Code generation | Lombok |

## Quick Start

```bash
# Clone and rename for your project
git clone <this-repo> my-service
cd my-service

# Run everything with Docker Compose
docker compose up --build

# App: http://localhost:8080
# Health: http://localhost:8080/actuator/health
# DB:   localhost:5432
```

For local development without Docker, start PostgreSQL separately and run:

```bash
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/appdb \
SPRING_DATASOURCE_USERNAME=app \
SPRING_DATASOURCE_PASSWORD=app \
./mvnw spring-boot:run
```

## Project Structure

```
agents/
  orchestrator.md      <- Start here: full feature pipeline (design -> code -> tests -> review)
  rest-api.md          # Generate REST endpoints + DTOs + validation
  code-review.md       # Review Java code quality
  test-generator.md    # Generate unit + integration tests
  microservice.md      # Design a new microservice
  mcp/servers.json     # MCP server config (Claude Code / Cursor)
  skills/
    spring-boot.md     # Spring Boot conventions used by all agents
    testing.md         # Testing conventions
    docker.md          # Docker conventions
docs/
  ai-usage.md          # How to use the agents effectively
docker/
  Dockerfile           # Multi-stage production build
src/
  main/
    java/.../Application.java
    resources/application.yml
  test/
    java/.../
      BaseIntegrationTest.java   # Shared Testcontainers base
      ApplicationTests.java
compose.yml            # Local dev environment
pom.xml
```

## Adapting This Template

### 1. Rename the project

Update these values throughout:

| File | Field | Replace with |
|---|---|---|
| `pom.xml` | `groupId` | your group (e.g. `com.acme`) |
| `pom.xml` | `artifactId` | your service name |
| `pom.xml` | `description` | short service description |
| `application.yml` | datasource defaults | your DB name/credentials |
| Java packages | `com.github.gescof.springbootaidevtemplate` | your package |

### 2. Generate your first feature with AI

Use the Orchestrator agent — it runs the full pipeline end-to-end:

```
# Using agents/orchestrator.md
Implement a Customer order management feature.
A Customer has: id, name, email, createdAt.
An Order has: id, customerId, totalAmount, status (PENDING/CONFIRMED/SHIPPED).
Endpoints: place order, confirm order, get by id, list by customer.
```

The orchestrator delegates to the other agents in sequence: REST API generation, test generation, then code review — with quality gates between each step.

### 3. Configure your environment

All sensitive values are externalised via environment variables:

```bash
SPRING_DATASOURCE_URL=jdbc:postgresql://host:5432/mydb
SPRING_DATASOURCE_USERNAME=myuser
SPRING_DATASOURCE_PASSWORD=secret
SERVER_PORT=8080
```

Defaults in `application.yml` are suitable for local Docker Compose usage.

## Testing

```bash
# Unit + integration tests (Testcontainers spins up PostgreSQL automatically)
./mvnw verify

# Tests only, skip build
./mvnw test
```

Integration tests extend `BaseIntegrationTest`, which manages a real PostgreSQL container via Testcontainers. No mocks, no fakes — the tests hit an actual database.

## AI Agents

See [docs/ai-usage.md](docs/ai-usage.md) for full usage examples.

| Agent | File | Purpose |
|---|---|---|
| **Orchestrator** | `agents/orchestrator.md` | Full feature pipeline: design -> code -> tests -> review |
| REST API | `agents/rest-api.md` | Generate complete CRUD endpoints (standalone) |
| Test Generator | `agents/test-generator.md` | Write unit + integration tests |
| Code Review | `agents/code-review.md` | Review Java code quality |
| Microservice | `agents/microservice.md` | Design a new service |

Use the Orchestrator as your default entry point for new features.

### Claude Code (MCP)

If you use Claude Code, the `agents/mcp/servers.json` file configures available MCP servers. See [docs/ai-usage.md](docs/ai-usage.md) for setup.

## Docker

```bash
# Build and start everything
docker compose up --build

# Production image only
docker build -f docker/Dockerfile -t my-service .
docker run -p 8080:8080 \
  -e SPRING_DATASOURCE_URL=jdbc:postgresql://host:5432/db \
  -e SPRING_DATASOURCE_USERNAME=user \
  -e SPRING_DATASOURCE_PASSWORD=secret \
  my-service
```

## Recommended Next Steps

- **Schema migrations**: Add [Flyway](https://flywaydb.org/) and switch `ddl-auto` from `update` to `validate`
- **API docs**: Add `springdoc-openapi-starter-webmvc-ui` for a Swagger UI at `/swagger-ui.html`
- **Security**: Add `spring-boot-starter-security` + JWT or OAuth2
- **Observability**: Add Micrometer + Prometheus scraping via Actuator
- **CI/CD**: Add a GitHub Actions workflow that runs `./mvnw verify`
