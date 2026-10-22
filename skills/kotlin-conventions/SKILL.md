---
name: kotlin-conventions
description: Kotlin and Android build, test and lint commands plus idiomatic Kotlin rules. Use when writing, reviewing or verifying Kotlin code (build.gradle.kts, *.kt, Android modules).
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
