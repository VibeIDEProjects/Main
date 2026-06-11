# ТЗ: NotificationService — Микросервис массовых уведомлений

## 1. Описание проекта

Асинхронный микросервис массовых уведомлений (SMS / Email / Push) с гарантиями доставки (at-least-once + идемпотентность), приоритезацией транзакционного трафика и подключаемой архитектурой брокеров (Kafka / RabbitMQ) и шлюзов.

В репозитории две параллельные реализации с идентичным API-контрактом:

- **Python 3.12 / FastAPI / SQLAlchemy (async)** — `src/python/`, production-ready, 31/31 автотестов зелёные.
- **Laravel 11 / PHP 8.3 / Eloquent / PDO** — `src/laravel/`, reference-реализация.

## 2. Цель

Спроектировать и реализовать распределённый сервис рассылки, который:

- Принимает запросы на массовую отправку по каналам SMS / Email / Push.
- Гарантирует доставку сообщений через Transactional Outbox + retry + DLQ.
- Защищён от дублей на стороне сервиса (двухуровневая идемпотентность).
- Приоритезирует транзакционный трафик над маркетинговым.
- Запускается ровно одной командой `docker compose up` для полного стека.

## 3. Стек технологий

### 3.1. Python-реализация (основная)

| Компонент | Версия / Название |
|-----------|-------------------|
| Язык | Python 3.12+ |
| Веб-фреймворк | FastAPI |
| ORM | SQLAlchemy (async) + Alembic |
| Брокер сообщений | Apache Kafka (по умолчанию) / RabbitMQ (через `NOTIFICATIONS_BROKER`) |
| Кэш / идемпотентность / rate-limit | Redis |
| БД | PostgreSQL 15 |
| Шаблонизатор | Jinja2 (опционально, `NOTIFICATIONS_TEMPLATES_ENABLED`) |
| Метрики | Prometheus (`/metrics`) |
| Тесты | pytest + aiosqlite (in-memory) |
| CI | GitHub Actions (ruff + pytest) |

### 3.2. Laravel-реализация (reference)

| Компонент | Версия / Название |
|-----------|-------------------|
| Язык | PHP 8.3 |
| Фреймворк | Laravel 11 |
| ORM | Eloquent + PDO |
| Брокер | Kafka / RabbitMQ (через ENV) |
| БД | PostgreSQL 15 |
| Кэш | Redis |
| Worker | `php artisan notifications:dispatch --watch` |

### 3.3. Общая инфраструктура

- PostgreSQL 15
- Redis 7
- Apache Kafka 3.x (KRaft) и/или RabbitMQ 3.x
- Kafka UI (опциональный профиль `ui`)
- Nginx (внешний reverse-proxy, вне `docker-compose`)

## 4. Функциональные требования

### 4.1. Массовая рассылка
- API `POST /api/v1/notifications` принимает канал, приоритет, массив получателей и payload.
- Поддерживаются каналы: `sms`, `email`, `push` (последний — через `NOTIFICATIONS_CHANNELS`).

### 4.2. Приоритезация трафика
- 3 очереди: `transactional` / `critical` / `marketing` с разным параллелизмом.
- Транзакционные сообщения (OTP, срочные изменения) обгоняют маркетинговые.

### 4.3. Детализация статусов доставки
- `GET /api/v1/recipients/{id}/notifications` — история и текущий статус по получателю.
- Жизненный цикл: `queued → sent → delivered` (success) / `failed → notifications.dlq` / `cancelled` (для scheduled).

### 4.4. Гарантии доставки (Reliability)
- **Transactional Outbox** — запись в `notifications` + `outbox` в одной транзакции, фоновая публикация в брокер.
- **At-least-once** — модель доставки, exactly-once на уровне бизнес-логики.
- **Двухуровневый retry** — exponential backoff внутри consumer'а + DLQ для poison messages.

### 4.5. Дедупликация (Idempotency)
- Заголовок `Idempotency-Key` (Redis) + `payload_hash` (sha256) в БД.
- Повторный запрос с тем же ключом → закэшированный ответ.

### 4.6. Шаблоны
- `NOTIFICATIONS_TEMPLATES_ENABLED=true` — Jinja-шаблоны в `payload: {_template, _context}` / `{_templates, _context}`.
- Защита от `__dunder__` / `__import__` / `eval` / `exec` через regex-фильтр.
- `jinja2.StrictUndefined` — отсутствующая переменная = ошибка.

### 4.7. Rate limit
- Два алгоритма: `fixed_window` (по умолчанию) и `token_bucket` (Redis Lua, рекомендуется в проде).
- Лимиты: `NOTIFICATIONS_RATE_LIMIT_RECIPIENT_PER_MIN`, `NOTIFICATIONS_RATE_LIMIT_PROVIDER_PER_SEC`.

### 4.8. Webhook клиенту
- Fire-and-forget POST на `NOTIFICATIONS_WEBHOOK_URL` при переходе `sent → delivered`.
- HMAC-SHA256 подпись в `X-Notification-Signature`.
- Retry при 5xx (exponential backoff, `NOTIFICATIONS_WEBHOOK_MAX_RETRIES`).

### 4.9. Cancel scheduled
- `POST /api/v1/notifications/{id}/cancel` — только для `queued`, идемпотентный `200 noop` на повторе, `409` на `sent`/`failed`.

### 4.10. DLQ CLI
- `python -m app.tools.dlq_cli {inspect,retry,purge}` — операционная инспекция / ретрай / очистка dead-letter queue.

## 5. Нефункциональные требования

- **Cloud Native**: упаковка в Docker-образы, запуск одной командой `docker compose up`.
- **Observability**: Prometheus-метрики (`notifications_sent_total`, `notification_webhook_dispatch_total`, `notification_rate_limited_total`, `provider_send_duration_seconds`).
- **Версионирование API**: URL-versioning `/api/v{major}/...`, активны 2 мажорные версии (N + N-1), deprecation period 6 месяцев, заголовки `Deprecation` / `Sunset` (RFC 8594). Подробнее — ADR-0002.
- **Абстракция брокера**: `MessageBroker` Protocol, переключение Kafka / RabbitMQ одной ENV. Подробнее — ADR-0001.
- **Тестирование**: 31/31 unit + integration тестов, в т.ч. IT (in-memory стабы Kafka/Redis, SQLite).
- **CI**: GitHub Actions — ruff + pytest + `scripts/final_check.py`.

## 6. Модели данных (основные)

### Notification
| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID | Уникальный идентификатор |
| channel | enum | `sms` / `email` / `push` |
| priority | enum | `transactional` / `critical` / `marketing` |
| status | enum | `queued` / `sent` / `delivered` / `failed` / `cancelled` |
| recipient_id | UUID | Получатель |
| payload | JSONB | Тело сообщения (или отрендеренный шаблон) |
| payload_hash | string | sha256 от payload (идемпотентность) |
| idempotency_key | string? | Значение `Idempotency-Key` |
| scheduled_at | datetime? | Для отложенных уведомлений |
| sent_at | datetime? | Момент передачи провайдеру |
| delivered_at | datetime? | Момент подтверждения доставки |
| created_at | datetime | Дата создания |
| updated_at | datetime | Дата обновления |

### Outbox
| Поле | Тип | Описание |
|------|-----|----------|
| id | PK | Идентификатор outbox-записи |
| notification_id | FK | Ссылка на `Notification` |
| payload | JSONB | Готовое сообщение для брокера |
| published_at | datetime? | NULL → ожидает публикации |
| attempts | int | Счётчик попыток |

### DLQ (`notifications.dlq`)
| Поле | Тип | Описание |
|------|-----|----------|
| id | PK | |
| notification_id | FK | Исходное уведомление |
| last_error | text | Причина фейла |
| failed_at | datetime | |

## 7. API Endpoints

| Метод | Путь | Назначение | Auth |
|-------|------|------------|------|
| GET | `/health/live` | Liveness probe | публичный |
| GET | `/health/ready` | Readiness probe (БД / Redis / брокер) | публичный |
| POST | `/api/v1/notifications` | Создать уведомление (202) | `X-API-Key` |
| GET | `/api/v1/notifications/{id}` | Получить статус | `X-API-Key` |
| POST | `/api/v1/notifications/{id}/delivered` | Подтверждение доставки провайдером | `X-API-Key` |
| POST | `/api/v1/notifications/{id}/cancel` | Отменить scheduled-уведомление | `X-API-Key` |
| GET | `/api/v1/recipients/{id}/notifications` | История получателя | `X-API-Key` |
| GET | `/metrics` | Prometheus-метрики | публичный |
| GET | `/docs` | Swagger UI (FastAPI автогенерация) | публичный |
| GET | `/redoc` | ReDoc | публичный |

Полная спецификация — `docs/openapi.yaml` (OpenAPI 3.0.3, 6 paths).
Postman-коллекция для импорта — `docs/postman-collection.json`.

## 8. Архитектура

```
                                 ┌──────────────────┐
HTTP/HTTPS ──▶ FastAPI (app) ──▶ │   PostgreSQL     │  notifications + outbox
                                 └──────────────────┘
                                          │
                                  outbox-publisher
                                          │
                                          ▼
                                 ┌──────────────────┐
                                 │  Kafka / RabbitMQ │  3 priority queues
                                 └──────────────────┘
                                          │
                                       worker
                                          │
                                 ┌──────────────────┐
                                 │   Redis          │  idempotency, rate-limit
                                 └──────────────────┘
                                          │
                                  provider gateway (mock / real)
                                          │
                                          ▼
                              POST /webhook (HMAC-SHA256)
```

## 9. Деплой

| Компонент | Где | Статус |
|-----------|-----|--------|
| `app` (FastAPI) | docker compose | ✅ |
| `worker` (NotificationConsumer) | docker compose | ✅ |
| `outbox-publisher` | docker compose | ✅ |
| PostgreSQL 15 | docker compose | ✅ |
| Redis 7 | docker compose | ✅ |
| Kafka 3 (KRaft) | docker compose | ✅ |
| RabbitMQ 3 | docker compose (опционально) | ✅ |
| Kafka UI | профиль `ui` | ✅ |
| Миграции | `alembic upgrade head` | ✅ |

## 10. Статус выполнения (Python-стек)

| Задача | Статус |
|--------|--------|
| CRUD/статусы уведомлений (queued / sent / delivered / failed) | ✅ Готово |
| Двухуровневая идемпотентность | ✅ Готово |
| Transactional Outbox | ✅ Готово |
| Приоритезация (3 очереди) | ✅ Готово |
| Retry + DLQ | ✅ Готово |
| DLQ CLI (inspect / retry / purge) | ✅ Готово |
| Плагин-архитектура брокеров (Kafka + RabbitMQ) | ✅ Готово |
| Jinja-шаблоны | ✅ Готово |
| Webhook клиенту (HMAC-SHA256, retry) | ✅ Готово |
| Rate limit (fixed_window + token_bucket) | ✅ Готово |
| Cancel scheduled | ✅ Готово |
| Prometheus-метрики | ✅ Готово |
| X-API-Key auth | ✅ Готово |
| 31/31 тестов (pytest) | ✅ Готово |
| URL-версионирование API + ADR-0002 | ✅ Готово |
| Laravel reference-реализация | ⏳ Отстаёт (push, delivered-webhook, scheduled_at, X-API-Key — см. `roadmap.md` P-блок) |

## 11. Локальный запуск (Python)

```bash
# 1. Подготовить .env
cp projects/NotificationService/src/python/.env.example \
   projects/NotificationService/src/python/.env

# 2. Поднять стек (PostgreSQL + Redis + Kafka + RabbitMQ + app + worker + outbox-publisher)
cd projects/NotificationService
docker compose up -d

# 3. Дождаться старта (10–15 секунд) и накатить миграции
docker compose exec app alembic upgrade head

# 4. Проверить готовность
curl -s http://localhost:8080/health/ready | python -m json.tool
```

После запуска:
- API: `http://localhost:8080`
- Swagger UI: `http://localhost:8080/docs`
- ReDoc: `http://localhost:8080/redoc`
- RabbitMQ management: `http://localhost:15672` (notifications / notifications)
- Kafka UI: `docker compose --profile ui up kafka-ui` → `http://localhost:8081`
- Prometheus метрики: `http://localhost:8080/metrics`

## 12. Ссылки

- **Репозиторий (Python + Laravel)**: https://github.com/VibeIDEProjects/NotificationService
- **ТЗ проекта**: `projects/NotificationService/docs/tz.md`
- **Архитектура**: `projects/NotificationService/docs/architecture.md`
- **Roadmap**: `projects/NotificationService/docs/roadmap.md`
- **Project Status (срез)**: `projects/NotificationService/docs/PROJECT_STATUS.md`
- **Реестр ENV**: `projects/NotificationService/docs/env.md`
- **OpenAPI 3.0.3**: `projects/NotificationService/docs/openapi.yaml`
- **Postman-коллекция**: `projects/NotificationService/docs/postman-collection.json`
- **ADR-0001 (абстракция брокера)**: `projects/NotificationService/docs/decisions/0001-broker-abstraction.md`
- **ADR-0002 (версионирование API)**: `projects/NotificationService/docs/decisions/0002-api-versioning.md`
