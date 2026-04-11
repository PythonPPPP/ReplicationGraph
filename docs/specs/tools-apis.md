# Tools / APIs Specification

## 1. Назначение

Слой Tools / APIs инкапсулирует доступ к внешним сервисам: прежде всего к LLM providers, а также к graph store, object store и sandbox execution runtime. Все внешние вызовы проходят через унифицированные адаптеры.

## 2. LLM Gateway

### 2.1 Контракт запроса

- `request_id`
- `run_id`
- `task_type`
- `model_class` (`critical|standard|economy`)
- `schema_id`
- `messages`
- `temperature`
- `timeout_ms`
- `max_retries`
- `fallback_policy`
- `idempotency_key`

### 2.2 Контракт ответа

- `provider`
- `model`
- `latency_ms`
- `usage_prompt_tokens`
- `usage_completion_tokens`
- `finish_reason`
- `parsed_output`
- `schema_valid`
- `error_code` при неуспехе

### 2.3 Правила отказоустойчивости

- retry только для retryable-кодов;
- backoff экспоненциальный с jitter;
- circuit breaker по provider/model;
- provider switch допускается только после исчерпания локальных retry;
- неподтверждённый schema-invalid ответ не считается успехом.

### 2.4 Ограничения

- жёсткий timeout обязателен;
- на один stage допускается конечное число provider switches;
- hedging разрешён только для критических коротких запросов и отключён по умолчанию из-за стоимости;
- максимальный размер контекста ограничен policy.

## 3. Graph Store Adapter

### Контракт

- `read_revision(revision_id)`
- `begin_revision(parent_revision_id)`
- `upsert_entities(batch)`
- `upsert_relations(batch)`
- `commit_revision()`
- `rollback_revision()`

### Надёжность

- optimistic concurrency control;
- commit атомарный на уровне ревизии;
- rollback обязателен при частичной ошибке записи.

## 4. Artifact/Object Store

### Контракт

- запись артефакта по `artifact_id` и `revision`;
- чтение по `artifact_id/revision`;
- checksum verification;
- immutable storage для завершённых ревизий.

## 5. Sandbox Runtime

### Контракт

- `job_id`
- `resource_profile`
- `entrypoint`
- `timeout_sec`
- `memory_limit_mb`
- `cpu_limit`
- `network_policy`
- `filesystem_policy`

### Ограничения безопасности

- исходящая сеть запрещена по умолчанию;
- доступ только к временному каталогу прогона;
- максимальное число процессов ограничено;
- side effects вне sandbox запрещены.

## 6. Error taxonomy

- `TIMEOUT`
- `RATE_LIMIT`
- `PROVIDER_UNAVAILABLE`
- `SCHEMA_INVALID`
- `AUTH_FAILURE`
- `DEPENDENCY_UNAVAILABLE`
- `POLICY_DENIED`
- `SANDBOX_LIMIT_EXCEEDED`

## 7. Метрики

- success/failure rate по каждому адаптеру;
- p95 latency;
- fallback activation rate;
- timeout rate;
- schema-invalid rate;
- sandbox job failure rate.
