---
name: kotlin-backend-persistence
description: >-
  Реализация и диагностика relational persistence в Kotlin/Spring: JPA/Hibernate entities, SQL, N+1/fetch plans, constraints, locking, pagination и migrations. Используй при изменении доступа к БД; JPA-specific правила не применяй к JDBC, jOOQ или R2DBC.
---
# Persistence в Kotlin Backend

## Порядок работы

1. Определи DB engine/version, persistence stack, migration tool и границу транзакции по проекту. JPA-разделы ниже применяй только при наличии JPA.
2. Для дефекта воспроизведи значимое поведение: SQL/query count, нарушение constraint, неверную страницу или concurrent update. Если среда недоступна, отдели подтверждённое от гипотезы.
3. Выбери минимальное изменение запроса, mapping, ограничения или транзакции; проверь влияние на callers и совместимость схемы.
4. Проверь результат на соответствующем DB engine, когда важна его семантика; сравни измерения до/после для performance fixes. Не запускай миграции на общей/production БД ради проверки.

При ревью выдай находки без изменения кода. При реализации сообщи исправление, проверки и ограничения проверки. Учитывай инструкции целевого проекта; этот skill работает без AGENTS.md из набора.

## JPA entities

Не используй Kotlin `data class` для JPA entities. Осознанно учитывай generated IDs, proxying, lazy associations, constructor requirements и nullability в persistence lifecycle. Используй подходящую Kotlin JPA/no-arg поддержку проекта.

## Equality и hashing entities

Не включай mutable relationships/collections в `equals`/`hashCode`. Осторожно с generated IDs, меняющимися с `null` на значение после persist. Equality strategy должна оставаться корректной в нужных приложению transient/managed/detached states. Не копируй универсальный рецепт без учёта proxies и ID lifecycle.

## DB constraints и indexes

Истинные integrity rules, подверженные race conditions, обеспечивай в БД: uniqueness, FK, nullability, check constraints и другие доступные БД invariants. Индексы добавляй по реальным access patterns/query plans, а не по naming conventions.

## Repositories

Экспонируй операции, реально нужные application layer. Не превращай repository в universal generic DAO. Выбирай derived queries, JPQL, Criteria, native SQL, projections, jOOQ и др. по сложности запроса и conventions проекта.

## Fetch plans и N+1

Диагностируй N+1 по реальному SQL/query count/logging/profiling. Используй целевые стратегии: `@EntityGraph`, explicit join fetch, batch fetching, DTO/projection queries. Осторожно сочетай collection fetch joins с pagination. Не переключай всё на `EAGER`.

## Relationships

Поддерживай обе стороны bidirectional association, если это требуется in-memory model. Cascade operations и `orphanRemoval` не взаимозаменяемы. Избегай огромных uncontrolled aggregate graphs.

## Persistence context

Lazy loading может сработать из logging, `toString`, JSON serialization, debugger или mapping вне transaction. Bulk JPQL/SQL updates обходят dirty checking и могут оставить context stale — clear/refresh при необходимости. Не завязывай web serialization на случайный Open Session in View.

## Locking и versions

При `read -> decision -> write` над общим состоянием application pre-check не гарантирует integrity. Выбери DB constraint, atomic update, lock или isolation по реальному требованию.

Используй `@Version` для optimistic concurrency, когда нужно обнаруживать lost updates. Pessimistic locking — только когда consistency/contention оправдывают blocking/deadlock tradeoffs. Ориентируйся на реальные isolation guarantees конкретной БД.

## Pagination

Всегда задавай deterministic ordering. Ограничивай page size публичных/дорогих запросов. Для больших или быстро меняющихся datasets рассматривай keyset/cursor pagination, когда offset становится дорогим/нестабильным. Проверяй generated SQL, если fetch strategy может нарушить page semantics.

## Migrations

Migration tool проекта (обычно Flyway/Liquibase) — source of truth схемы. Для rolling deployments предпочитай expand/migrate/contract, если breaking schema change нельзя атомарно применить ко всем instances. Не удаляй старую структуру, пока старые версии/consumers от неё зависят.

## Persistence tests

Для dialect, constraints, locking, migrations и SQL semantics используй реальный DB engine. Делай flush/clear, когда тест должен проверять БД, а не first-level cache.
