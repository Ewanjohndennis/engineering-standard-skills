---
name: java-standards
description: Apply Java-specific engineering standards when writing, reviewing, refactoring, or debugging Java code. Use alongside code-generation-standards and code-review-standards — this skill adds Java idioms, modern Java patterns, and language-specific pitfalls that the generic skills can't cover. Triggers on any Java file, class, method, or snippet.
---

# Java Standards

> Language-specific standards for Java. Apply on top of the generic code-generation-standards and code-review-standards skills — this narrows them to Java idioms, modern Java features (Java 11+), and common Java pitfalls.

---

## Modern Java — use it

Default to modern Java idioms. If the project targets Java 11+, there's no reason to write Java 8 style:

- Use `var` for local variables when the type is obvious from the right-hand side. Don't use `var` when it obscures what the type actually is.
- Use records for plain data carriers instead of manual POJOs with getters, equals, hashCode, and toString.
- Use sealed classes and interfaces to model closed hierarchies — they make exhaustive `switch` expressions possible and signal intent clearly.
- Use `switch` expressions (arrow form) instead of `switch` statements with fall-through where the result is a value.
- Use text blocks for multi-line strings (SQL, JSON templates, HTML snippets).
- Use `instanceof` pattern matching: `if (obj instanceof String s)` instead of casting after checking.

If the project is on Java 8 or an older LTS for a real reason, defer to that — but don't assume old Java out of habit.

---

## Nullability

`NullPointerException` is the most common Java bug. Treat nullability as a contract that must be explicit:

- Annotate parameters, return types, and fields with `@NonNull` / `@Nullable` (from whatever annotation library the project uses — Jetbrains, Checker Framework, Jakarta, etc). Be consistent within a file.
- Return `Optional<T>` from methods that may not produce a value — don't return `null` from public methods. Don't use `Optional` as a field type or parameter type; it's for return values.
- Never call `.get()` on an `Optional` without a guard. Use `.orElse()`, `.orElseGet()`, `.orElseThrow()`, or `.ifPresent()`.
- Use `Objects.requireNonNull()` at the top of constructors and public methods to fail fast with a clear message rather than throwing a confusing NPE later.

---

## Immutability

Prefer immutable objects. Make fields `final` by default; only make them mutable if there's a clear reason.

Use `Collections.unmodifiableList()` / `Map.copyOf()` / `List.of()` when returning collections from public methods — don't hand out internal mutable state. Use `List.of()`, `Map.of()`, `Set.of()` for truly immutable collections created at compile time.

For value objects, implement `equals()`, `hashCode()`, and `toString()` — or use records, which do this automatically. If a class is mutable but only partially, document clearly which parts are mutable and whether it's thread-safe.

---

## Exception handling

Use checked exceptions for recoverable conditions that callers are expected to handle. Use unchecked exceptions (`RuntimeException` subclasses) for programming errors and conditions the caller cannot reasonably recover from.

Don't catch `Exception` or `Throwable` broadly except at top-level boundaries (e.g. a servlet filter, a thread's uncaught exception handler). At those boundaries, log the full stack trace before swallowing.

Never swallow exceptions silently:
```java
// Bad
try { ... } catch (IOException e) { }

// Bad — loses the original cause
try { ... } catch (IOException e) { throw new RuntimeException("failed"); }

// Good — preserves cause chain
try { ... } catch (IOException e) { throw new RuntimeException("failed to read config", e); }
```

Use try-with-resources for anything that implements `AutoCloseable` — files, streams, connections, prepared statements. Never close resources manually in a `finally` block when try-with-resources is available.

Define custom exception classes for domain errors. Put them in a dedicated package or near the code that raises them. Name them clearly: `PaymentDeclinedException`, not `PaymentException`.

---

## Collections and streams

Use the most specific interface type for variables and parameters: `List`, `Map`, `Set` — not `ArrayList`, `HashMap`. Return interface types from public methods; accept interface types as parameters.

Use streams for transformations on collections — `filter`, `map`, `collect`, `reduce`. Don't use streams for simple indexed loops or when mutation is needed — a plain for-each is clearer. Don't chain more than three or four stream operations without introducing an intermediate variable with a meaningful name.

Use `Collectors.toUnmodifiableList()` / `toUnmodifiableMap()` when the result should not be modified by callers. Don't collect to a list and then wrap it in `unmodifiableList()` — use the collector directly.

Prefer `Map.getOrDefault()`, `Map.computeIfAbsent()`, and `Map.merge()` over the pattern of get-null-check-put.

---

## Concurrency

Mark shared mutable state explicitly. Use `synchronized`, `volatile`, `AtomicXxx`, or a higher-level concurrent collection (`ConcurrentHashMap`, `CopyOnWriteArrayList`) — don't assume thread safety where it hasn't been established.

Prefer `java.util.concurrent` over raw `synchronized` blocks for anything non-trivial: `ExecutorService`, `CompletableFuture`, `CountDownLatch`, `Semaphore`. Use `CompletableFuture` for async pipelines rather than callback chains.

`CompletableFuture.get()` without a timeout will block forever if the upstream task hangs. Always use `get(timeout, unit)` or `.orTimeout()` at boundaries where unbounded blocking is a real risk.

Document thread-safety guarantees (or lack thereof) on classes that are intended to be shared. `@ThreadSafe` and `@NotThreadSafe` (from `net.jcip.annotations` or equivalents) make intent clear.

---

## Design

Favour composition over inheritance for sharing behavior. Use interfaces with default methods for mixin-style behavior. Reserve abstract classes for when a genuine partial implementation is shared.

Keep constructors simple — avoid complex logic, I/O, or anything that can throw in a constructor. Use factory methods or builders for complex object construction.

Use the builder pattern when a constructor has more than three or four parameters, especially when some are optional. Builders are standard in Java; a telescoping constructor with seven parameters is not.

---

## Testing

Use JUnit 5. Use `@BeforeEach` and `@AfterEach` over JUnit 4's `@Before`/`@After`. Use `@ParameterizedTest` with `@MethodSource` or `@CsvSource` for table-driven tests.

Use AssertJ for assertions — it produces far clearer failure messages than plain JUnit assertions and reads more naturally. Avoid Hamcrest unless the project already uses it.

Use Mockito for mocking. Mock at the boundary — mock external services and repositories, not internal helpers. Verify interactions only when the interaction itself is the behavior under test, not just as a way to confirm code ran.

Test class names: `FooTest` for unit tests, `FooIT` for integration tests (if the project separates them with Maven Failsafe or equivalent).

---

## Common pitfalls

- **`equals()` on arrays**: `array1.equals(array2)` checks reference equality. Use `Arrays.equals()` or `Arrays.deepEquals()`.
- **String comparison**: always use `.equals()`, never `==`. Intern or `==` comparisons on strings work by accident due to the string pool and will fail on runtime-constructed strings.
- **Integer cache**: `Integer.valueOf(127) == Integer.valueOf(127)` is true; `Integer.valueOf(128) == Integer.valueOf(128)` is false. Never use `==` on boxed types.
- **`HashMap` with mutable keys**: if an object used as a map key is mutated after insertion, the map silently breaks. Keys must be effectively immutable.
- **`Calendar` and `Date`**: don't use them in new code. Use `java.time` (`LocalDate`, `ZonedDateTime`, `Instant`, `Duration`). For persistence, use `Instant` or `OffsetDateTime`.
- **`printStackTrace()`**: don't call it directly in production code. Log via the project's logging framework with appropriate context.
- **Catching `InterruptedException` and not re-interrupting**: if you catch `InterruptedException` and don't rethrow it, call `Thread.currentThread().interrupt()` to restore the interrupted status.
