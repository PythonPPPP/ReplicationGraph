# Agent / Orchestrator Specification

## 1. Назначение

Orchestrator управляет жизненным циклом run, а агенты выполняют специализированные задачи в пределах явно определённых контрактов. Оркестратор отвечает за правильный порядок стадий, лимиты, retry/fallback и checkpoint/recovery.

## 2. Стадии

- `INGEST`
- `EXTRACT`
- `NORMALIZE`
- `PLAN`
- `GENERATE`
- `VERIFY`
- `PACKAGE`

Дополнительные статусы:

- `WAIT_RETRY`
- `DEGRADED`
- `PARTIAL_COMPLETE`
- `FAILED`

## 3. Правила переходов

- переход на следующую стадию допускается только после прохождения entry/exit-check текущей;
- при retryable error stage переходит в `WAIT_RETRY`;
- при non-retryable error оркестратор либо активирует fallback, либо завершает run частично;
- переход `GENERATE -> VERIFY` запрещён, если не пройден minimal fact completeness gate.

## 4. Stop conditions

Run должен завершаться, если выполнено хотя бы одно условие:

- достигнута стадия `COMPLETE`;
- достигнут hard budget limit;
- исчерпаны retry и fallback;
- нарушен policy guard;
- потерян обязательный checkpoint без возможности восстановления.

## 5. Retry policy

- retry bounded;
- retry budget отдельный для stage и для run;
- на повторе запрещено менять входной revision незаметно;
- повтор шага обязан быть идемпотентным.

## 6. Fallback policy

### Для EXTRACT

- уменьшение batch size;
- переключение на secondary model;
- sparse-only retrieval при отказе dense/rerank.

### Для PLAN

- переход на более дешёвую модель допускается только при наличии подтверждённого KG revision;
- при отсутствии критических фактов stage завершает run частично.

### Для GENERATE

- генерация только для plan items с достаточным обоснованием;
- optional artifacts могут быть пропущены при budget pressure.

### Для VERIFY

- safety-check и consistency-check обязательны;
- расширенные эвристики качества допускается отключить в degraded mode.

## 7. Контракты агентов

Каждый агент возвращает:

- `agent_name`
- `run_id`
- `stage`
- `input_revision_refs[]`
- `output_payload`
- `schema_valid`
- `issues[]`
- `next_action_hint`

## 8. Метрики

- stage transition latency;
- retry count by stage;
- fallback count by stage;
- budget exhaustion rate;
- checkpoint restore rate;
- partial completion rate.
