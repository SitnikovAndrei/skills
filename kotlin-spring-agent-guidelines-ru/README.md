# Kotlin/Spring Agent Guidelines

Production-oriented инструкции для coding agents в Kotlin + Spring backend-проектах.

## Структура

```text
AGENTS.md
skills/
  architecture/SKILL.md
  kotlin/SKILL.md
  spring/SKILL.md
  persistence/SKILL.md
  testing/SKILL.md
  security/SKILL.md
  code-review-refactoring/SKILL.md
```

`AGENTS.md` — единый источник истины для общих правил проекта. Skills — специализированные расширения и намеренно **не повторяют** baseline-правила. Они используются вместе с `AGENTS.md`.

Skill-файлы следуют формату Agent Skills: YAML frontmatter с `name` и `description`, затем инструкции для задачи.

## Размещение в проекте

Держи `AGENTS.md` в корне репозитория. Skills скопируй или подключи в каталог, поддерживаемый coding agent (например `.claude/skills/` для Claude Code), если нужна автоматическая загрузка.

## Когда загружать skill

| Задача | Skill |
|---|---|
| Module boundaries, package-by-feature, modular monolith, ports/events | `architecture` |
| Kotlin API, types, nullability, coroutines, money/time | `kotlin` |
| Spring Boot web/services, DI, transactions, configuration, events | `spring` |
| JPA/Hibernate, SQL, fetch plans, pagination, locking | `persistence` |
| Unit/integration/Testcontainers/architecture/coroutine tests | `testing` |
| Authentication, authorization, JWT, webhooks, trust boundaries, abuse controls | `security` |
| Review, smells, performance risks, безопасный refactoring | `code-review-refactoring` |

Загружай только skills текущей задачи. `AGENTS.md` остаётся always-on baseline и должен присутствовать при использовании project skills. Если правило общее для нескольких skills, определяй его в `AGENTS.md`, а не копируй в каждый `SKILL.md`.

## Философия

Нет жёстких лимитов размера классов/методов и обязательных паттернов. Базовый подход — package-by-feature + modular boundaries. Hexagonal/DDD/CQRS вводятся только когда реально уменьшают coupling или complexity.

## Правило единого источника

Общие правила находятся в `AGENTS.md`. `SKILL.md` содержит только task-specific детали своей области. Если skill нуждается в baseline-правиле, он применяет `AGENTS.md`, а не повторяет его. Это уменьшает контекст и предотвращает рассинхронизацию.
