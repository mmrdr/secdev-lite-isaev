# S04 - DFD (шаблон)

Этот файл - **шаблон минимальной DFD** для семинара S04.
Скопируйте и ведите у себя в репозитории в: `SEMINARS/S04/S04_dfd.md`.

**Задача:** за 15-20 минут построить **понятную DFD** уровня сервиса (3-5 узлов) с явными **границами доверия** и подписями ключевых **потоков данных**. Эту DFD вы далее используете в `S04_stride_matrix_template.md` для STRIDE per element и приоритизации **L×I (1-5)**.

---

## Правила минимальной DFD

* **3-5 узлов** максимум: *Клиент/Front*, *API/Controller*, *Service/Бизнес-логика*, *DB/Хранилище*, *(опционально)* *External API/Queue*.
* Явно отметьте **trust boundaries** (Интернет ↔ Сервис, Сервис ↔ Внешние системы, Сервис ↔ Хранилище).
* На **рёбрах** подпишите **типы данных**: `JWT`, `PII`, `file`, `payment`, `DTO`, `SQL`, `event`, …
* Не уходим в микросервисы и детали реализации - уровень **контур/сервис**.
* Сразу держите связь с S03: около потока/узла можно указать **`[NFR: …]`** (ID из реестра S03).

---

## Базовый каркас (mermaid)

> Замените названия узлов под свой контекст, добавьте/уберите узлы, подпишите типы данных на рёбрах.
> **Важно:** границы доверия оформлены как `subgraph` с единым стилем.

```mermaid
flowchart LR
  %% --- Trust boundaries ---
  subgraph Internet[Интернет]
    U[Клиент]
  end

  subgraph Service[OlympGuide API]
    A[API Gateway]
    S[File & Export Service]
    D[(PostgreSQL)]
  end

  subgraph External[Внешние провайдеры]
    X[External Vendor API]
  end

  %% --- Основные потоки ---
  %% Пользователь -> API
  U -- "JWT + multipart/form-data (file) [NFR: NFR-001, NFR-002, NFR-003]" --> A
  U -- "JWT + HTTPS (GET /api/export) [NFR: NFR-004, NFR-005, NFR-006]" --> A

  %% Контроллер -> Сервис
  A -->|"Validated DTO"| S

  %% Сервис -> БД
  S -->|"ORM"| D

  %% Сервис -> Внешний API
  S -->|"gRPC [NFR: NFR-007, NFR-008, NFR-009]"| X

  %% --- Обратные потоки ---
  X -->|"JSON"| S
  S -->|"DTO response (CSV/JSON)"| A
  A -->|"HTTPS"| U

  %% --- Границы доверия ---
  classDef boundary fill:#f6f6f6,stroke:#999,stroke-width:1px;
  class Internet,Service,External boundary;
```

---

## Описание элементов DFD

### Узлы (Nodes)

| Узел | Описание | Trust Boundary |
|------|----------|----------------|
| **U (Клиент)** | Пользовательское приложение | Internet |
| **A (API Gateway)** | Точка входа, валидация, аутентификация | Service |
| **S (File & Export service)** | Основная бизнес-логика | Service |
| **D (PostgreSQL)** | Хранение данных | Service |
| **Q (Vendor API)** | Внешняя интеграция | External |

### Потоки данных (Data Flows)

| Поток | Тип данных | NFR связи | Описание |
|-------|------------|-----------|----------|
| **U → A** | JWT + multipart/form-data| NFR-001 (InputValidation) | Валидированные запросы |
| **U → A** | JWT + multipart/form-data| NFR-002 (InputValidation) | Валидированные запросы |
| **U → A** | JWT + multipart/form-data| NFR-003 (Rate Limiting) | Защита от DoS |
| **U → A** | JWT + HTTPS | NFR-004(Rate Limiting) | Защита от DoS |
| **U → A** | JWT + HTTPS | NFR-005(Privacy/PII) | Персональные данные |
| **U → A** | JWT + HTTPS | NFR-006(Perfomance) | SLO |
| **S → Q** | gRPC | NFR-007 (Observability) | Трассировка |
| **S → Q** | gRPC | NFR-008 (CircuitBreaker) | Стабильность работы сервиса |
| **S → Q** | gRPC | NFR-009 (Resilience-Timeout) | Защита от зависания |

---

## Как адаптировать под свой кейс

1. **Переименуйте узлы** (например, `A` → `Public API`, `S` → `Orders Service`, `D` → `PostgreSQL`).
2. **Добавьте/уберите** блоки: если нет внешних интеграций - удалите `External`; если есть очередь - раскомментируйте `Q`.
3. **Подпишите рёбра**: укажите **тип данных** (*JWT/PII/file/payment/DTO/SQL*) и ключевые **контроли из S03**:

   * Пример: `-- "PII [NFR: Privacy/PII, RFC7807]" -->`
   * Пример: `-- "file [NFR: InputValidation, Limits]" -->`
4. **Проверьте границы доверия**: всё, что «снаружи», должно явно пересекать границу *Internet → Service*; любые внешние интеграции - *Service → External*.

---

## Быстрые подсказки по подписанию потоков

* **Аутентификация:** `JWT/HTTPS`, [NFR: `Security-AuthN`, `RateLimiting`].
* **Авторизация/тенант:** упоминание tenant-id на внутренних ребрах, [NFR: `Security-AuthZ/RBAC`].
* **Валидация ввода/размер:** на публичных ребрах отметьте [NFR: `InputValidation`, `Limits`].
* **Ошибки/контракт:** для ответов от API добавьте [NFR: `API-Contract/Errors (RFC7807)`].
* **Приватность:** любые `PII` помечайте и связывайте с [NFR: `Privacy/PII`].
* **Наблюдаемость/аудит:** укажите `correlation_id` и audit-events на критичных переходах, [NFR: `Observability/Logging`, `Auditability`].
* **Timeout/Retry/CircuitBreaker:** на внешних вызовах, [NFR: соответствующие].

---

## Как это использовать дальше

* Перейдите к `S04_stride_matrix_template.md` и пройдите **STRIDE per element** (для каждого **узла и ребра**).
* У каждой угрозы укажите **NFR-ID** (связь с S03) и оцените **L×I (1-5)**; посчитайте `Score = L×I`.
* Выделите **топ-5** рисков - они уйдут в ADR (S05).

---

## Самопроверка (быстро)

* [ ] **3-5 узлов**, не больше.
* [ ] Границы доверия видимы и логичны.
* [ ] На рёбрах есть **типы данных** и пометки `[NFR: …]`.
* [ ] Ясно, где появляются `PII`, `JWT`, `file`, `payment`.
* [ ] Понятно, какие элементы/потоки пойдут в STRIDE и L×I (1-5).

---