# S03
## Панкратов Владислав, Титаренко Арсений, Исаев Кирилл 

---

## Поля реестра (data dictionary)

* **ID** - короткий идентификатор, например `NFR-001`.
* **User Story / Feature** - к какой истории/фиче относится требование (ссылка допустима).
* **Category** - выберите из банка (напр.: `Performance`, `Security-AuthZ/RBAC`, `RateLimiting`, `Privacy/PII`, `Observability/Logging`, …).
* **Requirement (NFR)** - *измеримое* требование (числа/пороги/границы действия).
* **Rationale / Risk** - зачем это нужно, какой риск/ценность покрываем.
* **Acceptance (G-W-T)** - проверяемая формулировка: *Given … When … Then …*.
* **Evidence (test/log/scan/policy)** - чем подтвердим выполнение (тест, лог-шаблон, сканер, политика).
* **Trace (issue/link)** - ссылка на задачу, обсуждение, артефакт.
* **Owner** - ответственный.
* **Status** - `Draft` | `Proposed` | `Approved` | `Implemented` | `Verified`.
* **Priority** - `P1 - High` | `P2 - Medium` | `P3 - Low`.
* **Severity** - `S1 - Critical` | `S2 - Major` | `S3 - Minor`.
* **Tags** - произвольные метки (через запятую).

---

## User Stories
## US-005 - Загрузка аватара/файла

* **Роль-Цель-Ценность:** Как пользователь, я хочу загрузить аватар, чтобы персонализировать профиль.
* **Кратко:** Проверка типа/размера, хранение под UUID.
l* **API/Endpoints:** `POST /api/files/avatar`
* **NFR hooks:** `InputValidation`, `RateLimiting`, `Storage-Quotas`, `API-Contract/Errors`

## US-014 - Экспорт данных (CSV/JSON)

* **Роль-Цель-Ценность:** Как пользователь, я хочу экспортировать свои данные, чтобы анализировать их вне системы.
* **Кратко:** Экспорт по фильтрам, контроль приватности, агрегации.
* **API/Endpoints:** `GET /api/export?format=csv|json`
* **NFR hooks:** `Privacy/PII`, `RateLimiting`, `Performance`, `API-Contract/Errors`

## US-018 - Интеграция с внешним API

* **Роль-Цель-Ценность:** Как система, я хочу запрашивать данные у внешнего API, чтобы обогащать информацию.
* **Кратко:** Пулы соединений, таймауты, кэш, деградации.
* **API/Endpoints:** `GET /api/integrations/vendor-x/*`
* **NFR hooks:** `Timeouts/Retry`, `CircuitBreaker`, `Performance`, `Observability/Logging`


## Таблица реестра

| ID      | User Story / Feature               | Category                  | Requirement (NFR)                                                                                         | Rationale / Risk                                                             | Acceptance (G-W-T)                                                                                                                                                                                | Evidence (test/log/scan/policy)                                                                    | Trace (issue/link) | Owner | Status | Priority    | Severity   | Tags |
|---------|------------------------------------|---------------------------|-----------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------| ------------------ | ----- | ------ | ----------- | ---------- | ---- |
| NFR-001 | US-005 - Загрузка аватара/файла    | Security-InputValidation  | Для `POST /api/files/avatar`: размер тела 15 ≤ MiB, extra поля запрещены                                  | Защита от больших данных                                                     | **Given** тело больше 15MiB и неизвестные поля<br>**When** POST `/api/files/avatar`<br>**Then** 413 с RFC7807, при неизвестных полях 400                                                          | test: `e2e-upload-limit`<br> test: `unit-test-upload-limit`<br> policy: schema/size                |                    |       | Draft  | P2 - Medium | S2 - Major |      |
| NFR-002 | US-005 - Загрузка аватара/файла    | Security-InputValidation  | Для `POST /api/files/avatar`: MIME только из allowlist                                                    | Защита от некорректного MIME type                                            | **Given** передан MIME type не image/jpeg или image/png<br>**When** POST `/api/files/avatar`<br>**Then** 413 с RFC7807, при неизвестных полях 400                                                 | test: `e2e-upload-type`<br> test: `unit-test-upload-type`<br> policy: schema/size                  |                    |       | Draft  | P2 - Medium | S2 - Major |      |
| NFR-003 | US-005 - Загрузка аватара/файла    | Rate Limiting             | На `POST /api/files/avatar` действует лимит 10 req/min на токен                                           | Защита от DoS                                                                     | **Given** активный токен <br>**When** выполняется 10+N запросов к `POST /api/files/avatar` за 60 секунд                                                                                                                                                                                                  | test: `e2e-rate-limit`, `unit-avatar-limit` log: `rate_limit_exceeded`                                                                                                   |                    |       | Draft  | P2 - Medium | S2 - Major |      |
| NFR-004 | US-014 - Экспорт данных (CSV/JSON) | Rate Limiting             | На `GET /api/export?format=csv\|json`действует лимит 10 req/min на токен                                  | Защита от DoS                                                                | **Given** активный токен <br>**When** выполняется 10+N запросов к `GET /api/export?format=csv\|json` за 60 секун <br>**Then**                                                                     | test: `e2e-rate-limit`<br> log: `rate_limit_exceeded`                                              |                    |       | Draft  | P2 - Medium | S2 - Major |      |
| NFR-005         | US-014 - Экспорт данных(CSV/JSON)                                   |  PII                          |  PII не попадает в логи                                                                                                         |  Приватность и комплаенс                                                                            |  **Given** DTO с персональными данными <br>**When** происходит логирование <br>**Then** поля PII маскированы/удалены                                                                                                                                                                                                 |  Пример логов с маскировкой                                                                                                  |                    |       | Draft  | P2 - Medium | S2 - Major |      |
| NFR-006 | US-014 - Экспорт данных(CSV/JSON) | Perfomance | P95 латентность для `GET /api/export?format=csv\|json` ≤ в течение 5 минут | SLO | **Given** сервис развернут и здоров <br>**When** на `GET /api/export?format=csv\|json` подается 20 RPS в течение 5 минут <br>**Then** P95 ≤ 300 и доля ошибок ≤ 1% | Отчёт нагрузочного теста | | | Draft  | P2 - Medium | S2 - Major |
| NFR-007 | US-018 - Интеграция с внешним API  | Observability/Logging     | Запросы к `GET /api/integrations/vendor-x/*` логируются в JSON и содержат correlation_id на каждом этапе  | Трассировка и разбор инцидентов                                              | **Given** запрос с X-Correlation-ID<br>**When** он проходит сервис<br>**Then** во всех логах появляется correlation_id и ключевые поля запроса/ответа                                             | log pattern: `json_struct`<br> test: `logging-correlation-id`<br> скрин поиска по `correlation_id` |                    |       | Draft  | P2 - Medium | S2 - Major |      |
| NFR-008 | US-018 - Интеграция с внешним API  | Resilience-CircuitBreaker | Circuit breaker срабатывает при более 50% ошибок за 1 мин при запросах `GET /api/integrations/vendor-x/*` | Стабильность работы сервиса при массовых неудачных обращениях к внешнему API | **Given** ≥50% ошибок `GET /api/integrations/vendor-x/*` за минуту<br>**When** обрабатываются новые запросы<br>**Then** Circuit breaker блокирует новые вызовы, вызовы фиксируются в логах        | client config: `cb`<br> test: `integration-cb-trigger`<br> log: `circuit_breaker_on`               |                    |       | Draft  | P2 - Medium | S2 - Major |      |
| NFR-009 | US-018 - Интеграция с внешним API  | Resilience-Timeout/Retry  | Исходящие вызовы к `GET /api/integrations/vendor-x/*`: timeout ≤ 2s, retry ≤ 3 с джиттером                | Защита от зависания при сбоях внешнего API                                   | **Given** недоступность `GET /api/integrations/vendor-x/*`<br>**When** сервис обращается к `GET /api/integrations/vendor-x/*`<br>**Then** суммарное ожидание ≤ 6s; не более 3 retry с джиттером   | client config: `timeout/retry`<br> test: `integration-unavailable-external-api`                    |                    |       | Draft  | P2 - Medium | S2 - Major |      |
---

## Памятка по заполнению

* **Измеримость.** В `Requirement` фиксируйте числа и границы (мс, RPS, минуты, MiB, коды 4xx/5xx, CVSS).
* **Проверяемость.** В `Acceptance (G-W-T)` используйте объективные условия и наблюдаемые факты (код ответа, квантиль, наличие заголовка, запись в лог).
* **Связность.** Сверяйте, чтобы NFR не конфликтовали (timeouts vs retry, rate limits vs SLO, privacy vs logging).
* **План проверки.** В `Evidence` укажите, чем это будет подтверждаться позже (test/log/scan/policy). В рамках семинара **реальные артефакты не требуются**.
* **Трассировка.** В `Trace` добавляйте ссылки на Issues/документы, чтобы потом не искать контекст.

---

## После семинара

* Перенесите/доработайте **8-10 утверждённых NFR** (по сути, те же строки) в раздел **NFR** вашего `GRADING/TM.md`.
* На S04-S05 свяжете эти NFR с угрозами (STRIDE) и ADR - по ID.
