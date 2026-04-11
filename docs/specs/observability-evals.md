# Observability / Evals Specification

## 1. Назначение

Компонент обеспечивает наблюдаемость выполнения, диагностику отказов и базовую оценку качества результата. Приоритет - стабильность системы и детектирование деградаций раньше, чем они повлияют на целостность итогового пакета.

## 2. Telemetry model

### 2.1 Метрики

Система собирает:

- run-level metrics;
- stage-level metrics;
- provider-level metrics;
- retrieval metrics;
- sandbox metrics;
- cost metrics.

### 2.2 Логи

Каждая запись должна включать:

- `timestamp`
- `run_id`
- `stage`
- `component`
- `severity`
- `event_type`
- `error_code` при наличии
- `revision_refs[]`

Логи не должны содержать секреты и лишние сырые данные.

### 2.3 Трейсы

Trace span обязателен для:

- stage execution;
- LLM calls;
- retrieval calls;
- graph store writes;
- sandbox jobs.

## 3. Алертинг

### Критические алерты

- provider outage;
- резкий рост timeout rate;
- длительно открытый circuit breaker;
- падение run success rate;
- рост schema-invalid rate;
- queue lag выше порога;
- рост sandbox limit exceeded.

### Предупреждающие алерты

- рост empty-hit rate retriever;
- рост unresolved items/run;
- повышение cost/run;
- увеличение доли degraded runs.

## 4. Evaluation checks

### 4.1 Runtime reliability checks

- доля успешных retry;
- доля успешных recovery с checkpoint;
- mean time to recovery по зависимостям;
- fallback effectiveness.

### 4.2 Output consistency checks

- fact-to-source grounding rate;
- plan-to-KG alignment rate;
- artifact-to-plan coverage;
- conflict detection recall на тестовом наборе.

### 4.3 Resource efficiency checks

- tokens per completed run;
- cost per completed run;
- repeated-call ratio;
- cache hit rate.

## 5. Минимальный dashboard

1. E2E reliability.
2. LLM provider health.
3. Queue/worker saturation.
4. Cost and token budget.
5. Retrieval quality proxy metrics.
6. Verification and conflict rates.

## 6. Retention и аудит

- сырые application logs имеют ограниченный retention;
- итоговые summaries и run manifests хранятся дольше;
- audit trail должен позволять восстановить причину `PARTIAL_COMPLETE` или `FAILED` статуса.
