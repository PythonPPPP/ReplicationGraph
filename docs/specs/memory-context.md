# Memory / Context Specification

## 1. Назначение

Компонент управляет состоянием прогона, долговременной памятью и сборкой контекста для LLM. Цель - минимизировать рассинхронизацию между шагами и не допускать переполнения контекста несущественными данными.

## 2. Уровни памяти

### 2.1 Run State

Хранит:

- текущую стадию;
- счётчики retry;
- budget usage;
- выбранные модели и provider health snapshot;
- ссылки на актуальные ревизии KG и артефактов.

### 2.2 Knowledge Memory

Хранит:

- сущности;
- связи;
- assumptions;
- unresolved items;
- conflicts;
- provenance.

### 2.3 Artifact Memory

Хранит:

- scaffold;
- generated modules;
- verification reports;
- package manifest.

## 3. Memory policy

- сырой вывод LLM не записывается как каноническая память без валидации;
- обновление KG разрешено только через schema-valid batch;
- conflicting updates создают новую запись конфликта, а не перетирают существующий факт;
- память должна быть версионированной и откатываемой.

## 4. Context assembly policy

Контекст собирается из следующих блоков:

1. системная инструкция этапа;
2. схема ожидаемого результата;
3. релевантный evidence-pack;
4. subset подтверждённых фактов;
5. только связанные unresolved items;
6. краткое состояние плана или артефакта, если нужно для этапа.

## 5. Context budget

- `hard_context_limit_tokens` фиксирован для model class;
- `reserve_output_tokens` обязателен;
- если бюджет превышен, сначала уменьшается число evidence items, затем применяются summaries к некритическому контексту;
- факты с высоким приоритетом безопасности и трассируемости не вытесняются из контекста summary-логикой.

## 6. Recovery

- после рестарта worker run state восстанавливается из checkpoint;
- незакоммиченные memory-updates отбрасываются;
- повтор шага возможен только с тем же `idempotency_key` или новой ревизией входа.

## 7. Метрики

- context size by stage;
- truncation rate;
- summary usage rate;
- stale-context incidents;
- recovery success rate.
