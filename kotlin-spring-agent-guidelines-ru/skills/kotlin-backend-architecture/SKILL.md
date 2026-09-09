---
name: kotlin-backend-architecture
description: >-
  Проектирование и ревью границ модулей Kotlin/Spring backend. Используй при изменении межмодульных зависимостей, выборе ports/events, декомпозиции или CQRS; не для каждой локальной правки Kotlin.
---
# Архитектура Kotlin Backend

## Контекст и результат

Изучи структуру затронутых модулей и существующие архитектурные проверки. Для локальной задачи не проводи аудит всей системы. Общий AGENTS.md применяй, если он существует в целевом проекте; этот skill не требует установки AGENTS.md из набора.

При проектировании объясни выбранную границу, направление зависимостей и существенный tradeoff. При реализации проверь изменённые зависимости доступными architecture checks и relevant tests. При ревью сообщай конкретные нарушения с местом в коде и последствием, не меняя файлы без запроса на исправление.

## Процесс архитектурного решения

1. Определи business capability и текущую границу модуля.
2. Определи, локально ли изменение или пересекает модули.
3. Сохраняй простейшую структуру, которая обеспечивает cohesion и явные зависимости.
4. Разделяй application/domain/infrastructure только когда сложность это оправдывает.
5. Вводи ports только когда dependency inversion создаёт полезную границу.
6. Переходи от логических feature-модулей к отдельным Gradle-модулям/сервисам только при необходимости более сильной изоляции.

## Форма feature-модуля

Для нового backend без иных требований предпочитай package-by-feature и modular monolith. Существующую разумную структуру не перестраивай попутно.

Небольшой feature может быть плоским: `OrderController`, `OrderService`, `OrderRepository`. Добавляй `api/application/domain/infrastructure` только когда они отражают реальные границы. Не создавай пустые зеркальные слои для trivial features.

## Публичный API модуля

Держи предоставляемый API бизнес-модуля намеренно небольшим. Публичными могут быть application facades/use cases, стабильные query interfaces, намеренно опубликованные domain/application contracts и integration-event contracts.

Repository implementations, persistence entities, provider DTOs, configuration details и internal orchestration должны оставаться внутри модуля.

Kotlin `internal` ограничивает compiler module, а не feature package. Если нужна compile-time isolation — рассмотри отдельные Gradle modules. Для логической изоляции используй Spring Modulith, Konsist, ArchUnit или аналогичные structural tests.

## Полезность интерфейса

Оценивай, сколько знаний требуется caller: не только методы и параметры, но и порядок вызовов, invariants, ошибки и настройки. Полезная граница скрывает существенную сложность и удерживает изменения локально; короткая сигнатура сама по себе этого не гарантирует.

Мысленно удали абстракцию, сохранив поведение. Если исчезают только переходы между обёртками, рассмотри её упрощение. Если правила, преобразования или координация разойдутся по callers, абстракция выполняет полезную работу. Сначала проверь скрытые контракты, включая transactions, authorization и lifecycle.

Это диагностический приём, а не команда удалять слой. Единственная реализация может оправдывать port, если он защищает реальную границу или направление зависимостей. Сохраняй принятую терминологию проекта.

## Hexagonal boundaries

Используй ports там, где application/domain не должен зависеть от изменчивого внешнего механизма. Типичные outbound ports: payment provider, CRM/external API, message publisher, repository при необходимости изоляции persistence, `Clock` для бизнес-значимого времени. Типичные inbound adapters: HTTP controllers, message consumers, scheduled jobs, CLI/admin entry points.

Не оборачивай стабильные in-process libraries только ради соответствия ports/adapters diagram.

## Ответственность слоёв

- **API / inbound adapter:** transport parsing/validation, protocol mapping, auth context, response/error representation.
- **Application:** orchestration use case, transaction/use-case boundaries, загрузка состояния, domain behavior, coordination ports, application result.
- **Domain:** бизнес-концепции, invariants, policies и решения без framework/I/O деталей.
- **Infrastructure / outbound adapter:** JPA/Hibernate, Redis, brokers, HTTP clients, filesystem, provider SDKs и mapping внешних представлений.

## Межмодульные зависимости

Предпочитай зависимости на стабильный API модуля: `payment -> order.OrderApi`. Не допускай `payment -> order.infrastructure.JpaOrderRepository` или `OrderEntity`.

Избегай циклов. Если два модуля требуют internals друг друга, пересмотри capability boundaries, выдели действительно общий стабильный concept или измени модель взаимодействия.

## CQRS/read models

Выделяй read models, когда query requirements существенно расходятся с write-domain: reporting/search projections, denormalized stores, независимо оптимизированные query models. Не вводи отдельную command/query infrastructure, если обычные application methods остаются ясными.

## Сигналы к усилению границ

Recurring cross-module leakage, provider-specific branching в application code, independent ownership/release pressure, крупные invariants с отдельным lifecycle, существенное расхождение read/write models, module tests с большим unrelated context.

## Проверка архитектуры

По возможности кодируй стабильные правила как executable checks: отсутствие циклов; импорт только разрешённых API соседнего модуля; domain не зависит от infrastructure/framework; controllers не обходят намеренную application boundary; запрещённые cross-feature imports ломают CI.

## Cross-module communication и delivery

Выбирай механизм по семантике:

- synchronous module API — caller нужен результат для завершения use case;
- event — публикуется business fact независимым consumers и допустима eventual consistency;
- durable messaging/outbox — эффект должен пережить process failure или перейти process/deployment boundary.

Не заменяй явные dependencies событиями ради сокрытия coupling. Publishing module владеет semantic meaning integration event; consumers не должны зависеть от его internal entities/persistence representation.

Обычная локальная DB transaction не обеспечивает атомарность с внешним HTTP call или публикацией в broker. Распределённая транзакция возможна только при поддержке и явном участии ресурсов; не предполагай её наличие. Для DB state + durable event publication рассматривай transactional outbox или другой явный consistency mechanism.

Для retries/duplicate delivery проектируй idempotency. Добавляй retry только после определения transient failures, retry owner, idempotency и limits/backoff.
