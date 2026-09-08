---
name: kotlin-spring-security-review
description: >-
  Реализация и ревью authentication, resource/tenant authorization, Spring Security, JWT, webhooks и обработки недоверенных данных в Kotlin/Spring. Используй для security review или изменений соответствующей границы доверия; обычный контроллер сам по себе не требует полного security audit.
---
# Security Review Kotlin + Spring

## Порядок работы и scope
1. Определи изменяемую операцию, principal, ресурс, tenant и входные недоверенные данные. При полном security review согласуй scope по запросу и доступным артефактам; локальную правку проверяй локально.
2. Проследи путь от входа до authorization check и опасной операции. Учитывай версии Spring Security, filter chains и способ передачи credentials.
3. Для существенной находки укажи файл/строку, предпосылку атаки, достижимый путь и последствие. Непроверенную гипотезу обозначь явно; отсутствие защитной аннотации само по себе не доказывает уязвимость.
4. В режиме исправления проверь разрешённый и запрещённый сценарии, включая чужой resource/tenant, если применимо. Используй локальные тесты; проверки живых систем требуют соответствующего разрешения.

Запрос на review означает отчёт без редактирования. Запрос на реализацию/исправление разрешает изменения в его scope. Не делай общий hardening попутно. Соблюдай инструкции целевого проекта; отдельный AGENTS.md из набора не требуется.

## Authentication и authorization
Рассматривай их отдельно. Для каждой защищённой операции определяй authenticated principal и конкретный resource/action, доступ к которому проверяется. Не доверяй tenant/user/resource IDs из запроса только потому, что caller authenticated.

## Spring Security
URL-level rules используй для coarse-grained endpoint policy, method/resource-level checks — когда доступ зависит от business object/action. Осознанно используй `@PreAuthorize` и учитывай proxy/interceptor semantics, включая self-invocation. Понимай порядок `SecurityFilterChain` и matcher precedence; избегай перекрывающихся правил с неочевидным итогом.

`401 Unauthorized` — отсутствующая/невалидная authentication; `403 Forbidden` — authenticated principal без нужного permission, если deliberate information-hiding policy не требует иного.

Пароли хешируй framework-supported password encoders; никогда plaintext или быстрыми general-purpose hashes. Для session auth используй secure cookie settings и protections against session fixation.

## Trust boundaries
HTTP params/headers/files, message payloads, webhooks, user-originated DB content и external-provider responses считай недоверенными до проверки для конкретного использования. Проверяй structure, size, allowed values и semantic constraints.

## Injection и path traversal
Используй parameterized SQL/JPQL. Не конкатенируй untrusted values в SQL, shell commands, templates, LDAP-like queries и другие interpreted languages. Для filesystem normalize/resolve path относительно разрешённой base directory и проверяй, что результат остаётся внутри неё.

## SSRF
По умолчанию не выполняй server-side requests на произвольные user-controlled URLs. Если dynamic destinations необходимы — ограничивай scheme/host/port, контролируй redirects и защищай internal/link-local/cloud-metadata destinations согласно threat model.

## Secrets и sensitive data
Используй secret-management проекта. Не логируй authorization headers, cookies, tokens, passwords, private keys, полные payment credentials и ненужные sensitive payloads. Минимизируй sensitive fields в API responses, events и audit logs.

## JWT и API keys
Проверяй JWT signature и claims, необходимые trust model: обычно issuer, audience, expiry/not-before и разрешённый algorithm/key source. Не доверяй decoded-but-unverified JWT. API keys не помещай в URLs/logs; хранение, сравнение и rotation — согласно threat model.

## Webhooks
Проверяй provider signature по точным bytes/fields, требуемым provider, а также timestamp/replay rules. Authenticate webhook до отметки processed. Учитывай duplicate delivery/retries.

## CORS и CSRF
CORS настраивай под реальные browser origins/methods/headers; не используй wildcard с credentials. Для cookie/session browser endpoints оценивай CSRF protection; не отключай CSRF глобально ради прохождения запроса. Для bearer APIs выбирай меры по способу transport/storage credentials.

## Error exposure
Не раскрывай stack traces, SQL, internal hostnames, credentials, provider secrets и лишнюю информацию о существовании accounts/resources. Ошибки auth должны быть полезны клиенту, но не облегчать enumeration.

## Abuse/resource controls
Для public/expensive endpoints рассматривай body/file limits, timeouts, pagination limits, concurrency/rate limits и operation-specific cost controls. Не допускай unbounded decompression, regex, collection reads, fan-out или file processing, управляемых attacker input. Слой rate limiting (app/gateway/edge) выбирай по topology и защищаемому ресурсу.

## Deserialization/dependencies
Избегай unsafe polymorphic deserialization attacker-controlled types. Предпочитай explicit schemas/types и conservative parser settings. Security-sensitive dependencies должны быть поддерживаемыми и patched по policy проекта.

## Security checklist
Проверь: principal identity; точную authorization resource/action; trust boundaries и dangerous sinks; injection/SSRF/path traversal; leakage secrets/data; replay/duplicates; resource exhaustion; auditable failures без раскрытия секретов.
