# opencode config

Персональная конфигурация [opencode](https://opencode.ai): агенты, команды, навыки
и базовые настройки сервера.

## Структура

```
opencode.jsonc   настройки opencode
agent/           роли (subagents)
command/         пользовательские команды
skills/          навыки с конвенциями по языкам
```

## Настройки (`opencode.jsonc`)

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "server": {
    "port": 4096,
    "hostname": "127.0.0.1"
  }
}
```

Сервер слушает только `127.0.0.1:4096`.

## Роли (`agent/`)

Роли запускаются как subagents через `task` и описаны в markdown с YAML frontmatter
(`description`, `mode`, `temperature`, `permission`).

| Файл | Роль | Права | Когда использовать |
|---|---|---|---|
| `agent/reviewer.md` | `reviewer` — строгий senior-ревьюер. | `edit: deny`, `bash: ask`, `task: deny` | После написания кода и перед мержем: ищет баги, проблемы безопасности и пробелы в требованиях. |
| `agent/verifier.md` | `verifier` — проверяет, что изменение реально работает. | `edit: allow`, `bash: allow`, `task: deny` | Перед объявлением задачи выполненной: сборка → тесты → линтеры, таблица «критерий → доказательство». |

Особенности:

- `reviewer` ничего не переписывает — только сообщает о проблемах. Находки
  группируются по серьёзности: **Blocker**, **Major**, **Minor**, **Nits**, плюс
  open questions и вердикт `approve` / `approve with nits` / `request changes`.
- `verifier` не правит production-код: при падении сообщает команду, вывод и
  `path:line`. Отсутствующие инструменты помечает как `unverified`, итог —
  `green` / `red` / `blocked`.

## Команды (`command/`)

| Команда | Файл | Агент | Назначение |
|---|---|---|---|
| `/spec` | `command/spec.md` | `plan` | Превращает черновое требование в проверяемую спецификацию: проблема, scope, критерии приёмки, затронутые области, дизайн, тест-план, риски. Код не меняет. |
| `/verify` | `command/verify.md` | `build` | Делегирует проверку субагенту `verifier` и кратко (до 5 строк) пересказывает вердикт. |

Обе команды принимают текст после команды через `$ARGUMENTS`.

Типичный цикл: `/spec <требование>` → реализация → `/verify <критерии>`.

## Навыки (`skills/`)

Навыки подключаются по совпадению задачи и содержат команды сборки/тестов/линта и
идиоматические правила. `AGENTS.md`, `Makefile` и CI проекта всегда важнее
дефолтов из навыка.

| Навык | Файл | Когда применять |
|---|---|---|
| `go-conventions` | `skills/go-conventions/SKILL.md` | Go-код: `go.mod`, `*.go`, `*_test.go`. |
| `kotlin-conventions` | `skills/kotlin-conventions/SKILL.md` | Kotlin/Android/Spring Boot: `build.gradle.kts`, `*.kt`, `application.yml`. |
| `python-conventions` | `skills/python-conventions/SKILL.md` | Python: `pyproject.toml`, `*.py`, `test_*.py`. |

Кратко по содержанию:

- **Go** — `go build/test -race/vet`, `gofmt`, `golangci-lint`; обёртка ошибок через
  `%w`, `errors.Is/As`, `context.Context` первым аргументом, без panic в библиотеках,
  `errgroup`, `slog`, table-driven тесты.
- **Kotlin** — только `./gradlew`; `val` вместо `var`, запрет `!!`, structured
  concurrency и внедряемые диспетчеры, Compose + `StateFlow`, detekt/ktlint.
- **Spring Boot** — constructor injection, `@ConfigurationProperties` вместо
  `@Value`, `@Transactional` на сервисах, DTO на границе и `ProblemDetail`,
  slice-тесты (`@WebMvcTest`, `@DataJpaTest`) и Testcontainers, Flyway/Liquibase
  для схемы, секреты только из env.
- **Python** — `uv`/`poetry`/pytest/ruff/mypy; type hints, без mutable default
  аргументов, `pathlib`, `logging`, `asyncio.to_thread` вместо блокирующего I/O.

`reviewer` и `verifier` делегируют языковую специфику соответствующему навыку.

## Что не в репозитории

`node_modules`, `package.json`, `package-lock.json`, `bun.lock` — игнорируются
через `.gitignore` (локальные зависимости, не часть конфигурации).
