# Spring Boot Conventions

These rules apply to all code generated for this project.

## Architecture

- **Layered architecture only**: Controller → Service → Repository. No skipping layers.
- Controllers handle HTTP concerns (request parsing, response status, validation trigger). No business logic.
- Services hold all business logic and own `@Transactional` boundaries.
- Repositories are interfaces only; no custom SQL unless Spring Data query methods are insufficient.

## Dependency injection

- Always use constructor injection. Never `@Autowired` on fields.
- Declare dependencies as `final` fields; use `@RequiredArgsConstructor` from Lombok.

## Data transfer

- Never serialize JPA entities to JSON. Always use dedicated response DTOs.
- Validate incoming data at the controller boundary using Jakarta Validation + `@Valid`.
- Use a `*Mapper` class or record to convert entity ↔ DTO. Keep mappers stateless.

## Transactions

- `@Transactional` on service write methods (create, update, delete).
- `@Transactional(readOnly = true)` on service read methods.
- Never put `@Transactional` on controllers or repositories.

## Error handling

- Throw domain-specific unchecked exceptions (e.g., `ResourceNotFoundException`).
- Handle exceptions centrally in a `@ControllerAdvice` class.
- Return `ProblemDetail` or a consistent error response DTO — never raw exception messages.

## Entities

- Use `@Getter` + `@Setter` (not `@Data`) on entities — `@Data` generates `equals`/`hashCode` that causes JPA problems.
- Use `@Builder` only on entities when no JPA lifecycle callbacks depend on the constructor.
- Prefer `Long` or `UUID` as entity ID type.

## General

- Prefer records for immutable DTOs (Java 21).
- Use `Optional` from repository methods; never return `null` from a service.
- Log at `INFO` for significant events, `DEBUG` for diagnostic detail. Never log passwords or tokens.
