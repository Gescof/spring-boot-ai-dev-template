# Docker Conventions

## Dockerfile

- Always use multi-stage builds: a `builder` stage with JDK and a `runtime` stage with JRE only.
- Base image: `eclipse-temurin:21-jdk` for build, `eclipse-temurin:21-jre` for runtime.
- Cache dependency resolution in a separate layer before copying source:
  ```dockerfile
  COPY .mvn/ .mvn/
  COPY mvnw pom.xml ./
  RUN ./mvnw dependency:go-offline -q   # cached unless pom.xml changes

  COPY src/ src/
  RUN ./mvnw clean package -DskipTests -q
  ```
- `EXPOSE` the application port. Use `ENTRYPOINT` (not `CMD`) for the Java process.
- Run as a non-root user when security requirements demand it.

## Docker Compose

- Pin image versions (e.g., `postgres:16`, not `postgres:latest`).
- Use `healthcheck` on the database so the app waits for it to be ready.
- Pass all configuration to the app via `environment` variables — never bake secrets into the image.
- Use named volumes for database data persistence.
- Use `depends_on` with `condition: service_healthy` (not just `depends_on: [postgres]`).

## Image hygiene

- Do not copy `target/`, `.git/`, or IDE config directories into the image. Add them to `.dockerignore`.
- Keep the runtime image as small as possible — only the JAR and JRE, nothing else.
- Tag images with a version or commit SHA in CI; avoid relying solely on `latest`.

## .dockerignore (recommended)

```
target/
.git/
.idea/
.vscode/
*.md
```
