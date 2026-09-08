---
name: kotlin-backend-code-review-refactoring
description: >-
  Ревью diff/PR и безопасный рефакторинг Kotlin/Spring backend: поиск дефектов, анализ ответственности и обоснованное упрощение. Используй по запросу на review/refactoring; ревью само по себе не разрешает изменять файлы.
---
# Code Review и Refactoring Kotlin Backend

## Режим и входные данные

- **Ревью:** изучи указанный diff/PR или код и необходимый контекст callers/tests. Не изменяй файлы. Если из запроса и репозитория нельзя определить объект ревью, уточни его.
- **Рефакторинг/исправление:** изменяй только запрошенный scope, сохраняя наблюдаемое поведение, кроме явно запрошенных behavior changes.

Сначала прочитай применимые инструкции проекта и выясни версии стека по build files, если вывод от них зависит. Этот skill не требует AGENTS.md из данного набора. Загружай доступные профильные skills только для затронутого риска, а не все сразу.

## Порядок ревью

1. Correctness и data integrity.
2. Security и failure handling.
3. Concurrency/transaction consistency.
4. Contract compatibility.
5. Architecture/coupling.
6. Readability/naming.
7. Duplication/cleanup.
8. Performance, если риск реалистичен/существенен.
9. Style concerns, не покрытые автоматикой.

Не начинай с косметики, пока остаются correctness risks.

## Severity и scope

- **must fix** — correctness, integrity, security, production failure, breaking contract;
- **should fix** — существенная maintainability/reliability проблема с практическим последствием;
- **optional** — улучшение, tradeoff которого зависит от будущего направления/предпочтений.

Соблюдай scope задачи. Не требуй unrelated redesign, если текущая структура не мешает безопасной реализации.

## Диагностика ответственности

Большой класс не автоматически god class. Ищи независимые clusters поведения/dependencies и разные reasons to change. Сильные сигналы: unrelated public operation families; зависимости разных subsystems; разные transaction/use-case lifecycles; конфликты несвязанных business changes; ответственность, которую можно назвать только через «и».

Дели по business responsibility/use-case boundaries, а не по размеру.

## Overengineering

Для цепочки `Controller -> UseCase -> Impl -> Service -> Impl -> Port -> Adapter -> Repository` спроси, чем владеет каждый уровень. Layer оправдан, если владеет policy, translation, lifecycle, isolation, module boundary, protocol semantics или meaningful variation. Иначе предлагай минимальное упрощение без изменения поведения.

## Underengineering

Ищи: unrelated behaviors в одном service; business decisions в transport/framework code; provider concerns по application logic; recurring cross-module leakage; повторяемые stable business concepts без владельца; global dumping-ground packages/modules. Новую boundary предлагай только если она снимает конкретное давление.

## Performance review

Сначала high-impact risks: algorithmic complexity на реалистичных объёмах; DB query count/plans; network round trips/fan-out; serialization volume; blocking на ограниченных execution resources; unbounded memory/queues/tasks; lock contention; повторная дорогая работа, которую имеет смысл amortize/cache только при безопасной cache semantics.

Не делай speculative micro-optimization без evidence или правдоподобного scale path.

## Generated output

Если проблема в generated code, найди owning schema/template/generator/source definition. При ревью адресуй замечание источнику; при запрошенном исправлении изменяй источник и регенерируй результат по правилам проекта.

## Безопасный refactoring

1. Зафиксируй текущее поведение тестами/наблюдениями.
2. Сделай одно structural change.
3. Запусти relevant tests/static/architecture checks.
4. Заверши, когда запрошенное структурное изменение выполнено и relevant checks пройдены. При регрессии исправь её в рамках своей правки; не продолжай декомпозицию ради самого процесса.

По возможности разделяй behavior changes и большие structural moves. Для рискованного legacy без тестов сначала создай characterization coverage изменяемых paths.

## Формат результата ревью

Для каждого существенного finding укажи severity, файл/строку и условие возникновения, затем: **Problem** — конкретный риск; **Impact** — production/maintenance consequence; **Smallest useful fix** — минимальное исправление; **Alternative** — только если другой design существенно меняет tradeoff. Не пиши generic «clean code» замечания без конкретного последствия.

Обосновывай findings достижимым execution path, контрактом или результатом проверки. Отделяй доказанные дефекты от гипотез. Не создавай замечания ради количества; если существенных проблем не найдено, скажи это и обозначь существенные ограничения проверки.
