# GitHub Copilot Instructions

This project is a Spring Boot 3 template for building production-ready REST APIs with Java 21 and PostgreSQL.

## Stack

- **Spring Boot 3.x** with Java 21 virtual threads
- **Spring Data JPA** + Hibernate + PostgreSQL
- **Jakarta Validation** for request validation
- **Spring Boot Actuator** for health and info endpoints
- **Lombok** for boilerplate reduction
- **JUnit 5** + Mockito + Testcontainers for testing

## Architecture

Follow strict layer separation: `Controller → Service → Repository → Entity`.

```
controller/    # HTTP concerns only — no business logic
service/       # Business logic — owns @Transactional boundaries
repository/    # JpaRepository interfaces — no custom SQL unless necessary
model/
  entity/      # JPA entities
  dto/         # Request and response DTOs (never serialize entities to JSON)
  mapper/      # Entity ↔ DTO conversion
exception/     # Domain exceptions + @ControllerAdvice handler
```

## Code style

### Dependency injection
```java
// CORRECT — constructor injection with Lombok
@Service
@RequiredArgsConstructor
public class OrderService {
    private final OrderRepository orderRepository;
}

// WRONG — field injection
@Service
public class OrderService {
    @Autowired
    private OrderRepository orderRepository;
}
```

### Transactions
```java
// Read operations
@Transactional(readOnly = true)
public OrderResponse findById(Long id) { ... }

// Write operations
@Transactional
public OrderResponse create(CreateOrderRequest request) { ... }
```

### DTOs
Use Java records for immutable DTOs:
```java
public record CreateOrderRequest(
    @NotNull Long customerId,
    @NotEmpty List<@Valid OrderItemRequest> items
) {}

public record OrderResponse(Long id, String status, BigDecimal total) {}
```

### Controllers
```java
@RestController
@RequestMapping("/api/v1/orders")
@RequiredArgsConstructor
@Validated
public class OrderController {

    private final OrderService orderService;

    @PostMapping
    public ResponseEntity<OrderResponse> create(@RequestBody @Valid CreateOrderRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED).body(orderService.create(request));
    }

    @GetMapping("/{id}")
    public ResponseEntity<OrderResponse> findById(@PathVariable Long id) {
        return ResponseEntity.ok(orderService.findById(id));
    }
}
```

### Entities
```java
@Entity
@Table(name = "orders")
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private Long customerId;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private OrderStatus status;
}
```

## Testing

- Unit tests: `@ExtendWith(MockitoExtension.class)` with `@Mock` / `@InjectMocks`
- Controller tests: `@WebMvcTest` with `@MockBean` service
- Integration tests: extend `BaseIntegrationTest` (Testcontainers PostgreSQL)
- Test naming: `methodName_whenCondition_thenExpectedOutcome`

## What to avoid

- `@Data` on JPA entities (generates broken `equals`/`hashCode`)
- `open-in-view: true` (already disabled — do not re-enable)
- Returning JPA entities from controller methods
- Business logic in controllers
- `@Autowired` on fields
- Hardcoded credentials or connection strings (use environment variables)
