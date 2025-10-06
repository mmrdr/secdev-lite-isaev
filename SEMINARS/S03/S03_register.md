# S03 - Шаблон реестра NFR (Given-When-Then)
## Панкратов Владислав, Титаренко Арсений, Исаев Кирилл 

Этот файл - рабочий **реестр NFR** на семинаре.
Заполняйте его у себя в репозитории в: `SEMINARS/S03/S03_register.md`.

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



| ID      | User Story / Feature                | Category                 | Requirement (NFR)                                                         | Rationale / Risk                  | Acceptance (G-W-T)                                                                                                                                | Evidence (test/log/scan/policy)                                                       | Trace (issue/link) | Owner | Status | Priority    | Severity   | Tags |
|---------|-------------------------------------|--------------------------|---------------------------------------------------------------------------|-----------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------| ------------------ | ----- | ------ | ----------- | ---------- | ---- |
| NFR-001 | US-005 - Загрузка аватара/файла     | Security-InputValidation | Для `POST /api/files/avatar`: размер тела 15 ≤ MiB, extra поля запрещены  | Защита от больших данных          | **Given** тело больше 15MiB и неизвестные поля<br>**When** POST `/api/files/avatar`<br>**Then** 413 с RFC7807, при неизвестных полях 400          | test: `e2e-upload-limit`; <br>test: `unit-test-upload-limit`<br> policy: schema/size  |                    |       | Draft  | P2 - Medium | S2 - Major |      |
| NFR-002 | US-005 - Загрузка аватара/файла     | Security-InputValidation | Для `POST /api/files/avatar`: MIME только из allowlist                    | Защита от некорректного MIME type | **Given** передан MIME type не image/jpeg или image/png<br>**When** POST `/api/files/avatar`<br>**Then** 413 с RFC7807, при неизвестных полях 400 | test: `e2e-upload-type`; <br>test: `unit-test-upload-type`<br> policy: schema/size    |                    |       | Draft  | P2 - Medium | S2 - Major |      |
| NFR-003 | US-014 - Экспорт данных (CSV/JSON)  | Rate Limiting            | На `GET /api/export?format=csv\|json`действует лимит 10 req/min на токен  | Защита от DoS                     | **Given** активный токен <br>**When** выполняется 10+N запросов к `GET /api/export?format=csv\|json` за 60 секун <br>**Then**                     |                                                                                       |                    |       | Draft  | P2 - Medium | S2 - Major |      |
|         |                                     |                          |                                                                           |                                   |                                                                                                                                                   |                                                                                       |                    |       | Draft  | P2 - Medium | S2 - Major |      |
|         |                                     |                          |                                                                           |                                   |                                                                                                                                                   |                                                                                       |                    |       | Draft  | P2 - Medium | S2 - Major |      |
|         |                                     |                          |                                                                           |                                   |                                                                                                                                                   |                                                                                       |                    |       | Draft  | P2 - Medium | S2 - Major |      |
|         |                                     |                          |                                                                           |                                   |                                                                                                                                                   |                                                                                       |                    |       | Draft  | P2 - Medium | S2 - Major |      |
|         |                                     |                          |                                                                           |                                   |                                                                                                                                                   |                                                                                       |                    |       | Draft  | P2 - Medium | S2 - Major |      |

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