---
name: kotlin-spring-backend
description: >-
  Реализация и диагностика Spring Boot backend на Kotlin: HTTP mapping/validation, DI, transactions/proxies, configuration, Spring events и HTTP clients. Используй при изменении поведения Spring; ORM fetch plans и SQL проверяй профильным persistence skill, если он доступен.
---
# Kotlin + Spring Backend

## Порядок работы

1. Определи версии Java, Kotlin и Spring Boot, MVC или WebFlux, используемый transaction manager и модель concurrency по build/config и соседнему коду. Не обновляй стек попутно.
2. Проследи затронутый путь от входной точки до application/adapter boundary; проверь proxy, validation и transaction behavior только там, где это влияет на задачу.
3. При запросе на реализацию/исправление внеси минимальное изменение и проверь relevant web slice или integration test, если корректность зависит от Spring. Для простого внутреннего изменения достаточно проверки его поведения.

При ревью не меняй файлы без запроса на исправление. В результате укажи выявленную причину или изменение и фактически выполненные проверки. Применяй инструкции целевого проекта; наличие AGENTS.md из набора не требуется.

## Dependency injection

Предпочитай constructor injection и явные зависимости. Не выставляй persistence entities как внешние API contracts по умолчанию.

## Controllers

Controllers отвечают за routing, request/response contracts, HTTP validation, интеграцию authentication/authorization, status/headers и mapping transport ↔ application. Решения use case держи ниже controller boundary.

## Validation

Jakarta Bean Validation используй для transport-level structural constraints. Request validation, business invariants и DB integrity — разные уровни защиты; один не заменяет другой.

## Application services

Application service оркестрирует coherent use case: загружает состояние, вызывает domain behavior, координирует external ports и возвращает application result. Здесь естественно размещать Spring transaction boundary, если она нужна use case.

## Transactions и proxies

Размещай `@Transactional` вокруг coherent use cases, а не произвольных repository calls. Помни proxy semantics: self-invocation может обходить `@Transactional`, method security и другие interceptors.

Не держи DB transaction открытой во время медленных network calls без осознанного consistency/failure tradeoff. `readOnly = true` — hint, поведение которого зависит от transaction manager и persistence provider; это не универсальная оптимизация и не гарантия запрета записи.

Проверяй rollback rules: по умолчанию `RuntimeException`/`Error` вызывают rollback, checked exceptions — нет; проект может переопределять это поведение. В Kotlin отсутствие checked exceptions на уровне языка не меняет правила Spring.

Учитывай propagation и фактический transaction manager: thread-bound транзакция не переносится автоматически в новый поток; reactive transaction зависит от Reactor context. Не считай `suspend` доказательством корректной передачи транзакции.

## Configuration

Для структурированной конфигурации предпочитай type-safe `@ConfigurationProperties` с validation вместо множества `@Value`. Environment-specific wiring держи на configuration/adapter boundary.

## Framework error mapping

Преобразуй application/domain failures в HTTP централизованно, например через `@ControllerAdvice`. Используй стабильное structured error representation; Spring HTTP types не должны проникать в domain.

## Spring events и transaction timing

Понимай, когда listener выполняется: немедленно, after commit, async или вне transaction. Фазы `@TransactionalEventListener` выбирай осознанно. In-process Spring events не дают durable delivery.

## Spring Modulith

Используй для документации/проверки modules, allowed dependencies, module tests и runtime observations, когда это полезно. Если границы важны, не полагайся только на package naming.

## HTTP clients

На adapter boundary настрой timeouts, serialization, authentication, error mapping, connection pooling и observability при необходимости. Provider-specific DTOs/errors держи внутри adapter.

## OpenAPI

Следуй принятой стратегии проекта: code-first или spec-first. Не меняй её попутно. При code-first документация должна отражать реальный transport contract/validation. При spec-first меняй исходную specification и регенерируй code, не редактируй generated output. При эволюции публичного API проверяй compatibility с consumers и rolling deployments.

## Kotlin annotations

Осознанно выбирай annotation use-site targets, особенно для Bean Validation, Jackson, JPA и reflection. Java-примеры field annotations нельзя механически переносить на Kotlin properties.

## Virtual threads

На поддерживаемых Java/Spring virtual threads подходят многим blocking request/response workloads с JDBC или blocking HTTP clients. Не вводи Reactor/coroutines только из-за blocking I/O, если virtual threads проще удовлетворяют требованиям. Но и не включай virtual threads вслепую: проверь library/thread-local/context assumptions и характер нагрузки.

## Observability

Сохраняй observability важных операций: structured logs, correlation/request IDs и meaningful business IDs; metrics/traces — когда помогают эксплуатации. Не жертвуй correctness ради logging, не логируй чрезмерно hot paths и не раскрывай secrets/sensitive payloads.
