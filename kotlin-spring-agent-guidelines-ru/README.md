# Kotlin/Spring Agent Guidelines

Семь самостоятельных Agent Skills для разработки и ревью Kotlin/Spring backend и необязательный шаблон [AGENTS.md](AGENTS.md) с общими соглашениями проекта.

## Состав

| Skill | Когда применять |
|---|---|
| [kotlin-backend-architecture](skills/kotlin-backend-architecture/SKILL.md) | Границы модулей, ports/events, CQRS, архитектурные проверки |
| [kotlin-language-design](skills/kotlin-language-design/SKILL.md) | Kotlin types, nullability, equality, коллекции, coroutines |
| [kotlin-spring-backend](skills/kotlin-spring-backend/SKILL.md) | Spring HTTP/validation, DI, transactions/proxies, config/events |
| [kotlin-backend-persistence](skills/kotlin-backend-persistence/SKILL.md) | JPA/SQL, N+1, constraints, locking, pagination, migrations |
| [kotlin-spring-testing](skills/kotlin-spring-testing/SKILL.md) | Выбор и диагностика unit/slice/integration/contract tests |
| [kotlin-spring-security-review](skills/kotlin-spring-security-review/SKILL.md) | Authentication, authorization и границы доверия |
| [kotlin-backend-code-review-refactoring](skills/kotlin-backend-code-review-refactoring/SKILL.md) | Ревью diff/кода и запрошенный рефакторинг |

## Установка

Копируй нужные каталоги целиком, сохраняя их имена. Имя каталога совпадает с YAML `name`; `SKILL.md` должен лежать непосредственно в нём.

Для Codex используй `.agents/skills/` целевого проекта либо `~/.agents/skills/` для персональной установки. Пример результата:

```text
your-project/
  .agents/skills/
    kotlin-backend-persistence/
      SKILL.md
    kotlin-spring-testing/
      SKILL.md
```

Сам каталог `skills/` этого репозитория служит для хранения исходников. Его наличие не означает, что агент автоматически обнаружил скиллы. Для другого агента используй поддерживаемый им каталог; не копируй весь набор одновременно в несколько сканируемых каталогов одного агента.

`AGENTS.md` устанавливать необязательно. Если нужны его соглашения, объедини подходящие разделы с инструкциями целевого проекта, сохранив существующие команды, ограничения и архитектурные решения. Не заменяй существующий файл целиком. Skills учитывают применимые инструкции проекта и не требуют именно нашего шаблона.

После установки проверь наличие нужных имён в списке skills агента и выполни реалистичный запрос с явным вызовом, например:

```text
Используй $kotlin-backend-persistence: исследуй N+1 в списке заказов,
сохрани семантику пагинации и проверь число SQL-запросов до и после.
```

Для проверки автоматического выбора повтори подходящий сценарий в новой сессии без явного имени skill. В Codex CLI/IDE список доступен через `/skills`; если изменения не появились, перезапусти Codex.

При обновлении старого набора заменяй ранее установленные каталоги `architecture`, `kotlin`, `spring`, `persistence`, `testing`, `security`, `code-review-refactoring` соответствующими именами из таблицы. Сначала сохрани локальные правки и убери старую копию из сканируемого каталога, чтобы не оставить дубликаты.

## Принципы применения

Подключай только области, необходимые задаче. Несколько skills допустимы, если задача пересекает их границы: например, fetch plan и его DB-тест. Наличие контроллера не требует полного security audit; локальная правка не требует архитектурного пересмотра.

Ревью возвращает замечания без редактирования. Реализация и исправление допускают изменения в запрошенном scope. Версии и команды проверок определяются по проекту, а недоступные проверки явно отмечаются в результате.

В `AGENTS.md` находятся общие соглашения; в skills — профильные правила и порядок работы. Короткое повторение важного ограничения допустимо для самостоятельной установки. Не добавляй references или scripts без конкретной пользы; текущие инструкции помещаются в SKILL.md.

## Проверка изменений

Проверяй YAML frontmatter, непустые `name`/`description`, соответствие имени каталогу, уникальность имён и существование относительных ссылок. Для формата можно использовать `skills-ref validate <каталог-skill>` или доступный `quick_validate.py` из skill-creator; отдельно проверяй ограничения, которые конкретный валидатор не покрывает.

Поведение проверяй по [сценариям](tests/scenarios.md). Успешная проверка YAML не доказывает правильный выбор skill или качество результата. Сценарии — набор для прогонов, а не заявление об уже пройденных тестах.

## Источники формата

- [Agent Skills specification](https://agentskills.io/specification)
- [OpenAI: Build skills](https://learn.chatgpt.com/docs/build-skills)

Русский язык инструкций не меняет требования к YAML и именам каталогов. `scripts/`, `references/`, `assets/` и `agents/openai.yaml` необязательны.
