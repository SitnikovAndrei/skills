---
name: kotlin-spring-testing
description: >-
  Создание, исправление и ревью тестов Kotlin/Spring: выбор test slice, Testcontainers, transaction boundaries, coroutine, messaging и contract tests. Используй для задач о тестах или когда изменение требует этих специальных проверок; не запускай полный аудит тестов при любой правке.
---
# Тестирование Kotlin + Spring

## Порядок работы
1. Определи наблюдаемый контракт или регрессию и существующие test conventions, framework versions и команды Gradle/Maven wrapper либо CI. Не придумывай имя task или путь модуля.
2. Выбери минимальный уровень проверки, который действительно обнаруживает ошибку. Для regression test по возможности убедись, что он падает на исходном дефекте и проходит после исправления.
3. Запусти относящиеся к изменению проверки. Если Docker, DB или другая зависимость недоступны, сообщи это; не выдавай mock-only проверку за подтверждение интеграции и не отключай тест ради зелёного результата.

В результате укажи, какое поведение проверено, какие команды выполнены и что осталось непроверенным. Соблюдай инструкции целевого проекта; AGENTS.md из этого набора не обязателен.

## Минимальный тест, доказывающий поведение
Используй pure unit tests для domain calculations/decisions; focused Spring slices для framework mapping; integration tests для DB, transactions, serialization, security, messaging и wiring; небольшое число E2E для наиболее важных flows. Не mock'ай то, чья корректность зависит от реального framework/database/protocol.

## Unit tests
Используй deterministic inputs/outputs. Для сложных invariants проверяй boundaries и сохранение инварианта; property-based testing полезен для больших input spaces, если делает правило яснее. Контролируй clock/randomness. Fakes/stubs часто лучше interaction-heavy mocks.

## Spring web tests
Проверяй routing/status, validation, serialization, error mapping и security integration. Domain/use-case behavior преимущественно тестируй ниже HTTP boundary.

## DB integration tests
Используй production DB engine через Testcontainers, когда важны dialect, constraints, locking, migrations или query semantics. Flush/clear context, если нужно увидеть реальное DB behavior. N+1/fetch изменения проверяй измерением запросов.

## Transaction-boundary tests
Automatic rollback может скрыть commit-time failures, transaction events, lazy loading и async side effects. Если важен commit/rollback timing — тест должен наблюдать реальную границу.

## Coroutine tests
Используй `kotlinx-coroutines-test` для virtual time/deterministic scheduling. Не используй реальные sleeps в unit tests. Проверяй cancellation/timeouts, если они часть контракта. Coroutine test не доказывает non-blocking underlying I/O.

## External HTTP
Используй stub servers/contract fixtures для serialization, headers, provider errors, timeouts и protocol expectations. Обычный CI не должен зависеть от нестабильных внешних сервисов.

## Messaging tests
Для durable messaging/outbox проверяй persisted outbox state/publication, commit ordering, duplicate delivery, redelivery, retry/DLQ routing и schema mapping. Crash/recovery scenarios тестируй, когда риск оправдывает сложность.

## Architecture tests
Spring Modulith/Konsist/ArchUnit подходят для no cycles, approved module APIs, domain→infrastructure restrictions и forbidden cross-feature imports. Не кодируй субъективный formatting/style как architecture tests.

## Надёжность тестов
Fixtures/builders должны снижать шум, не скрывая значимое поле. Избегай order dependence, shared mutable global state, uncontrolled randomness, wall-clock assumptions, sleeps-as-sync и retries, маскирующих flaky tests.
