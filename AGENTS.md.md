# Coding Guidelines — Matteo Rossi

## Identity & Context

All projects live under `it.matteoroxis.*` group ID. Every project is a standalone Spring Boot 3.x application targeting MongoDB Atlas for no-sql database and PostgreSQL for relational data. Code serves two purposes: production-quality implementation and demo material for technical articles on foojay.io or InfoQ.com.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 21–25 (use the version declared in the project's `pom.xml`) |
| Framework | Spring Boot 3.x (always via `spring-boot-starter-parent`) |
| Database | MongoDB (`spring-boot-starter-data-mongodb`) or PostgreSQL (`spring-boot-starter-data-jpa`) |
| AI / Embeddings | Spring AI 1.x (`spring-ai-bom`) |
| Reactive | Project Reactor (`Flux`, `Mono`) when the project is reactive |
| Build | Maven only — no Gradle |
| Lombok | Do not introduce it in other projects. |

---

## Architecture Patterns

### Layered MVC (apply as a default)

```
controller/      — @RestController, thin, delegates to service
service/         — business logic
repository/      — Spring Data interfaces
document/        — @Document classes (named XxxDocument)
domain/          — enums, value types, principal objects
dto/request/     — inbound DTOs
dto/response/    — outbound DTOs
mapper/          — hand-written mappers (toDocument / toResponse)
exception/       — typed exceptions + GlobalExceptionHandler
config/          — @Configuration beans
```

- Domain objects are framework-free. 
- Business rules live inside the service layer.
- Repository layer does not contain business logic — only data access methods.
- Controllers are thin, with no logic beyond request validation and delegation to services.
- Use `record` for immutable value objects (e.g., `Money`, `OrderId`, `ErrorResponse`).

### Hexagonal / Ports & Adapters 

```
domain/          — pure Java, no framework annotations
ports/           — interfaces only (use cases + repository ports)
adapters/        — controller, MongoDB adapter classes
adapters/documents/  — @Document classes
adapters/mapper/ — hand-written mapper (no MapStruct)
config/          — @Configuration, package-private classes
exception/       — domain-specific exceptions
```

- Domain objects are framework-free. Business rules live here (state machine via guard clauses in methods).
- Use `record` for immutable value objects (e.g., `Money`, `OrderId`, `ErrorResponse`).
- Configuration classes that wire adapters to ports are **package-private**.


### AI / RAG Projects 

- Controllers go in `api/` or `controller/`
- AI tools annotated with `@Tool(description = "...")` live in `component/` or `tool/`
- `VectorStore` is injected, never instantiated directly
- Use `TokenTextSplitter` for chunking; configure chunk size in the constructor, not via properties
- MCP tools: implement in a `@Component` class, wire via `McpServerConfiguration`

### Reactive Projects 

- Return `Flux<T>` / `Mono<T>` from service and repository methods — no blocking calls
- Use `ReactiveMongoRepository`
- Apply `.onBackpressureLatest()` only when the use case explicitly requires it

---

## Code Style

### Dependency Injection

Always use **constructor injection**. Never `@Autowired` on fields.

```java
// Correct
public class OrderService {
    private final OrderRepository repository;

    public OrderService(OrderRepository repository) {
        this.repository = repository;
    }
}
```

### Domain Objects

- No getters/setters in domain classes — use accessor methods with meaningful names (e.g., `order.id()`, not `order.getId()`).
- Enforce invariants in the constructor or in business methods with `IllegalStateException` / `IllegalArgumentException`.

```java
public void markAsShipped() {
    if (status != OrderStatus.PAID) {
        throw new IllegalStateException("Only paid orders can be shipped");
    }
    this.status = OrderStatus.SHIPPED;
}
```

### Document Classes

- Named `XxxDocument`, annotated with `@Document("collection_name")`.
- Use standard getters/setters (not records) — Spring Data needs them for deserialization.

### Exception Handling

- One `GlobalExceptionHandler` per project (`@RestControllerAdvice`).
- Typed exceptions per domain concept (`OrderNotFoundException`, `ForbiddenException`).
- Use `record ErrorResponse(int status, String message)` as the response body.

### Logging

- Use `LoggerFactory.getLogger(ClassName.class)` (SLF4J). No `System.out.println`.
- Log at `INFO` on ingestion/write operations. `WARN` for missing resources. No `DEBUG` spam.

---

## Testing

### Unit Tests (domain + service)

- JUnit 5 with `@ExtendWith(MockitoExtension.class)` for services.
- Domain tests use plain JUnit 5 assertions (`assertEquals`, `assertThrows`).
- Service tests use **AssertJ** (`assertThat`, `assertThatThrownBy`, `assertThatNoException`).
- `@Mock` + `@InjectMocks` — no `@SpringBootTest` for unit tests.
- `@DisplayName` on service tests with a short description of the scenario (Italian is fine).

### Test Method Naming

```
methodName_scenario_expectedOutcome()
// e.g.: getAllOrders_customer_succeeds
//        getOrderById_notFound_throws
```

### Test Structure (domain tests)

Use Given / When / Then comments:

```java
@Test
void shouldMarkOrderAsPaid() {
    // Given
    Order order = new Order(...);

    // When
    order.markAsPaid();

    // Then
    assertEquals(OrderStatus.PAID, order.status());
}
```

### Integration Tests

- Use `@SpringBootTest` + Testcontainers for MongoDB when integration coverage is needed.
- Mirror the main package structure under `src/test/java`.


---

## What NOT to Do

- Do not add Lombok to projects that don't already use it.
- Do not use `@Autowired` field injection.
- Do not return raw `Document` (BSON) from controllers — always map to a DTO.
- Do not add `System.out.println` in production code.
- Do not use `Optional.get()` without a preceding check — always use `orElseThrow()` with a typed exception.
- Do not create MapStruct mappers — hand-write them.
- Do not use `var` for complex or non-obvious types; prefer explicit types for readability.
- Do not add comments in English in code that already has Italian inline comments — keep the language consistent within a file.

---

## Build & Run

```bash
# Build
./mvnw clean package -DskipTests

# Run
./mvnw spring-boot:run

# Test
./mvnw test
```

Each project is self-contained. MongoDB connection string is always in `application.properties` under `spring.data.mongodb.uri`.
