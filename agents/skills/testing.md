# Testing Conventions

These rules apply to all tests generated for this project.

## Framework

- JUnit 5 for all tests. No JUnit 4.
- AssertJ for assertions (`assertThat`). Avoid bare `assertEquals`.
- Mockito for mocking (`@Mock`, `@InjectMocks`, `@Captor`). Use `@ExtendWith(MockitoExtension.class)`.

## Test naming

Use: `methodName_whenCondition_thenExpectedOutcome`

Examples:
- `save_whenEmailAlreadyExists_thenThrowDuplicateException`
- `findAll_whenRepositoryIsEmpty_thenReturnEmptyList`
- `update_whenIdNotFound_thenThrowNotFoundException`

## Unit tests

- Test one class in isolation; mock all its collaborators.
- One `@BeforeEach` for common setup; keep it short.
- One assertion per logical outcome (multiple `assertThat` on the same result is fine).
- Use `@ParameterizedTest` + `@MethodSource` for tables of inputs/outputs.
- Verify `Mockito.verify(...)` calls only when the interaction itself is the contract being tested.

## Controller tests (`@WebMvcTest`)

- Use `@WebMvcTest(YourController.class)` to load the web layer only.
- Mock the service with `@MockBean`.
- Send requests via `MockMvc`; assert status and JSON body.
- Test: valid payload → 2xx, invalid payload → 400, missing resource → 404.

## Integration tests

- Extend `BaseIntegrationTest` — do not create new Testcontainers configurations.
- Use `@Sql` or repository saves in `@BeforeEach` to set up DB state; clean up in `@AfterEach` or use `@Transactional` on the test.
- Test the full HTTP → Service → DB round-trip for critical flows.

## Coverage priorities

1. Service business logic and edge cases
2. Controller validation (bad input → 400)
3. Repository custom queries
4. Integration for critical paths (create, update, delete)

## What NOT to test

- Getters and setters
- Lombok-generated code
- Framework wiring (Spring already tests this)
