---
name: kotlin-conventions
description: Kotlin, Spring Boot and Android build, test and lint commands plus idiomatic Kotlin and Spring rules. Use when writing, reviewing or verifying Kotlin code (build.gradle.kts, *.kt, Android modules, Spring Boot services, @RestController, @ConfigurationProperties, application.yml).
---

# Kotlin conventions

Repo `AGENTS.md`, `Makefile` and CI config win over these defaults. Always use the
Gradle wrapper (`./gradlew`), never a system `gradle`.

## Commands

```sh
./gradlew build
./gradlew test
./gradlew testDebugUnitTest          # Android
./gradlew detekt                     # if configured
./gradlew ktlintCheck                # if configured
./gradlew :app:assembleDebug         # Android build
./gradlew lintDebug                  # Android lint
```

Run one test: `./gradlew test --tests "com.example.MyTest.method"`.

## Rules

- Prefer `val` over `var`; immutable data by default.
- Use `data class`, `sealed interface`/`sealed class` for closed hierarchies, `enum` for simple sets.
- Never use `!!` in production code; use `?.`, `?:`, `requireNotNull`, or validate at the boundary.
- Avoid platform types from Java interop leaking into Kotlin APIs; annotate Java returns.
- Coroutines: inject `CoroutineDispatcher`, never hardcode `Dispatchers.IO` in logic; keep `suspend` functions main-safe; cancel scopes with structured concurrency. Prefer `Flow` for streams, `StateFlow` for UI state.
- Android: no blocking work on the main thread; hoist state in Compose (`StateFlow` + `collectAsStateWithLifecycle`); use `ViewModel` for screen logic; `remember`/`derivedStateOf` correctly.
- Prefer scope functions only when they improve readability; avoid deep `let`/`run` nesting.
- Handle errors explicitly: `Result`, sealed error types, or `runCatching` with logging — never silently drop.
- Tests: JUnit5 (or JUnit4 in Android), `kotlin.test` assertions, `runTest` + `TestDispatcher` for coroutines, Turbine for `Flow`. Assert behavior.
- Public API changes and new dependencies: call them out; keep versions in the version catalog (`libs.versions.toml`) if the project uses one.

## Spring Boot

Detect Spring by `org.springframework.boot` in `build.gradle.kts` or `@SpringBootApplication`.
Keep the `kotlin-spring`/`all-open` plugin so proxied beans stay open.

### Commands

```sh
./gradlew bootRun
./gradlew bootJar
./gradlew test                        # unit + slice tests
./gradlew test --tests "*ControllerTest"
./gradlew check                       # tests + detekt/ktlint
./gradlew bootBuildImage              # if configured
```

Run one test: `./gradlew test --tests "com.example.UserControllerTest.shouldReturn404"`.

### Wiring

- Constructor injection only; no field injection and no `@Autowired` on a single constructor. Declare dependencies `private val`.
- Group config in `@ConfigurationProperties` + `@ConfigurationPropertiesScan` instead of scattered `@Value`. Validate with `@Validated` and jakarta constraints; use `@DefaultValue` carefully.
- Prefer `@Configuration(proxyBeanMethods = false)` plus explicit `@Bean`s over component scanning sprawl in library modules.
- Define bean conditions explicitly (`@ConditionalOnProperty`, `@Profile`); no hidden environment-dependent beans.

### Web layer

- `@RestController` returns DTOs, never JPA entities; map explicitly at the boundary.
- Validate input at the boundary: jakarta constraints on DTOs + `@Valid` on `@RequestBody`/`@RequestParam`. Kotlin non-null types do not protect against malformed external input.
- One `@RestControllerAdvice` maps domain errors to responses; prefer `ProblemDetail` (RFC 7807). Never leak stack traces, SQL or internal messages.
- Return proper status codes (`201` + `Location`, `404`, `409` for conflicts); do not encode failures in a `200` body.
- Bound page sizes and validate sort fields; never pass client-provided field names straight into queries.

### Persistence

- Keep `@Transactional` on public service methods, not controllers or repositories; mark read paths `readOnly = true`. Keep transactions short — no HTTP calls or long computation inside.
- Self-invocation bypasses the proxy: a private/internal call does not start a new transaction. Inject the bean or extract a collaborator instead.
- Do not use `data class` for JPA `@Entity` (generated `equals`/`hashCode` and `copy` break lazy proxies and identity). Use a normal class with an id-based `equals`.
- Prefer Spring Data derived queries or explicit `@Query`; avoid unbounded `findAll()` on exposed endpoints. Fetch lazily but watch N+1 — use projections or `@EntityGraph`.
- Migrations (Flyway/Liquibase) are required for schema changes; never rely on `ddl-auto=update` outside tests.

### Tests

- Slice first: `@WebMvcTest` + `MockMvc`/`MockMvcTester` for controllers, `@DataJpaTest` for repositories, `@JsonTest` for serialization. Use `@SpringBootTest` only for real integration.
- Use Testcontainers for the real database instead of H2 whenever the project already does; H2 hides SQL/dialect bugs.
- Spring Boot 3.4+: replace `@MockBean`/`@SpyBean` with `@MockitoBean`/`@MockitoSpyBean`.
- Avoid `@DirtiesContext` as a fix for shared state; isolate data per test (`@Transactional` rollback or cleanup) and use `@ActiveProfiles("test")` with `application-test.yml`.
- Assert HTTP status, body and side effects — not internal method calls. Use `MockMvcTester`/`WebTestClient` assertions over raw JSON string matching.

### Config & operations

- Secrets come from env vars or a secret store, never from `application.yml` committed to git. Use `application-{profile}.yml`; keep the default profile safe.
- Enable Actuator health/metrics where the project uses it; do not expose sensitive endpoints publicly.
- `@Async` needs `@EnableAsync` and a dedicated, bounded executor (do not use the default `SimpleAsyncTaskExecutor`). `@Scheduled` runs on every replica — use ShedLock or an external scheduler for singletons.
- Use Micrometer + structured `slog`/SLF4J logging; log correlation ids, never secrets, tokens or full request bodies.
- Version API paths and call out breaking contract changes; publish/regenerate OpenAPI when the project has it.
