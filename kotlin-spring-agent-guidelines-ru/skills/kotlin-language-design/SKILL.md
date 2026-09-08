---
name: kotlin-language-design
description: >-
  Написание и ревью Kotlin backend-кода, когда важны nullability, моделирование типов, equality, коллекции, coroutines или Java interoperability. Не используй для Android/Compose UI и задач, не затрагивающих Kotlin-код.
---

# Kotlin: язык и проектирование API

## Порядок работы

1. Изучи изменяемый API, его callers и версии Kotlin/JVM в build files. Сохраняй conventions и публичные контракты проекта.
2. Выбери только относящиеся к задаче правила ниже; не перерабатывай соседний код ради идиоматичности.
3. При изменении проверь компиляцию и затронутое поведение существующими проверками проекта. Добавляй тесты там, где они доказывают значимый контракт или предотвращают регрессию.

При ревью укажи конкретный риск и место в коде. При реализации кратко сообщи изменение и выполненные проверки. Следуй применимым инструкциям целевого проекта; AGENTS.md из этого набора не является обязательной зависимостью.

## Nullability

Используй nullability Kotlin напрямую вместо Java-подобного `Optional<T>` в обычных Kotlin API.

Избегай `!!`, кроме намеренного fail-fast для доказанного инварианта. Не используй nullable collections/booleans, если `null` не имеет отдельного смысла от empty/false.

Выбирай между safe calls, Elvis, явным ветвлением, `requireNotNull` и `checkNotNull` исходя из семантики. Устойчивые инварианты закрепляй при создании/валидации объекта, а не размазывай повторяющиеся проверки по коду.

## Моделирование типов

Используй `data class`, когда структурное равенство и `copy()` соответствуют семантике типа.

Для закрытых внутренних иерархий предпочитай exhaustive `when` без `else`, когда все варианты должны обрабатываться явно. Это не требует отвергать неизвестные значения внешнего протокола, допускающего расширение.

Используй `sealed interface` / `sealed class`, когда закрытая иерархия даёт полезную исчерпывающую обработку. Используй `enum class` для стабильного набора символических вариантов, которым не нужны специфичные для подтипов состояние или поведение.

Используй `@JvmInline value class` для строго типизированных идентификаторов/значений, если это предотвращает реальные классы ошибок и ограничения interoperability приемлемы.

Не создавай wrapper-типы для каждого primitive без конкретной пользы для корректности или домена.

## Деньги и время

Используй `BigDecimal` или явный money-тип для точной десятичной денежной арифметики. Правила округления и scale определяй на бизнес-границе, которой они принадлежат.

Выбирай типы `java.time` по семантике:

- `Instant` — абсолютный момент времени;
- `LocalDate` — календарная дата;
- `OffsetDateTime`/`ZonedDateTime` — когда offset/zone является частью контракта или бизнес-смысла;
- `LocalDateTime` — только локальное время, timezone-контекст которого явно задан в другом месте.

Не трактуй `LocalDateTime` без timezone как абсолютный момент. Инжектируй/контролируй `Clock`, если текущее время влияет на бизнес-поведение или тесты.

## Функции и ясность call site

Предпочитай early return, когда он яснее выражает предусловия и уменьшает вложенность.

Избегай ловушек boolean-параметров вроде `send(message, true, false)`. Если различие важно в месте вызова, используй enum/options object/отдельную операцию.

Выделяй функцию, когда у неё есть устойчивое понятное имя и она снижает когнитивную нагрузку; не дроби ясный последовательный код на мелкие обёртки.

## Scope functions

Используй `let`, `run`, `with`, `apply`, `also`, только когда семантика receiver/result остаётся очевидной.

Избегай вложенных scope functions с неоднозначными `it`/`this`. Прямой императивный код лучше хитрой цепочки scope functions.

## Extensions

Используй extensions для операций, которые концептуально принадлежат receiver и остаются дешёвыми/обнаруживаемыми.

Не присоединяй через extensions чужое business behavior к типу другого модуля в обход ownership.

Не скрывай в extensions дорогое I/O, неожиданные side effects или большие глобальные utility namespaces.

## Коллекции и sequences

Предпочитай `val` и read-only interfaces коллекций, когда мутация не нужна. `val` фиксирует ссылку, а `List` запрещает изменение через этот интерфейс; они не гарантируют неизменяемость объекта или его элементов. Если важен immutable snapshot, исключи изменяемые aliases либо используй подходящую persistent/immutable collection.

Используй преобразования коллекций, когда они ясно выражают намерение. Предпочитай простой цикл, если длинная цепочка скрывает control flow или существенны промежуточные аллокации.

Используй `Sequence`, когда lazy processing даёт реальную пользу; для небольших коллекций он не обязательно быстрее.

## Equality и identity

Проектируй равенство осознанно: value objects обычно используют structural equality; identity-bearing objects часто не должны определять equality по всем полям; mutable-поля не должны неожиданно ломать membership в hash-based collections.

Для ORM-managed entities учитывай generated IDs, proxies и lifecycle; при наличии используй `$kotlin-backend-persistence`. Не генерируй equality по всем полям entity.

## Coroutines

Используй structured concurrency и scopes, принадлежащие lifecycle. Не используй `GlobalScope` для прикладной работы.

Передавай cancellation дальше и не поглощай `CancellationException` широкими catch-блоками.

Используй `coroutineScope`, когда ошибка дочерней задачи должна завершить группу; `supervisorScope` — когда независимые ошибки детей намеренно изолированы.

`suspend` не превращает blocking I/O в non-blocking. Явно учитывай JDBC, filesystem, legacy clients и другую блокирующую работу.

Используй `async` только для реального параллельного выполнения, а не как универсальную обёртку обычных вызовов.

## Java/Spring interoperability

Используй минимальную практичную visibility. Kotlin `internal` относится к compiler module, а не package.

Учитывай final-классы Kotlin, Java nullability annotations, annotation use-site targets, reflection, proxying и no-arg requirements библиотек.

Предпочитай Kotlin compiler plugins для Spring/JPA вместо ручного объявления больших частей кодовой базы `open`.

## Virtual threads и coroutines

Для blocking Spring/JDBC workloads на поддерживаемых версиях Java/Spring virtual threads могут быть проще, чем внедрение coroutines или reactive chains только ради блокирующего I/O.

Не смешивай platform threads, virtual threads, coroutines и Reactor без причины. Выбери одну основную модель concurrency для execution path и понимай, как через границы проходят context propagation, blocking, cancellation и transaction semantics.
