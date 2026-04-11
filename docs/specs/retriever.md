# Retriever Specification

## 1. Назначение

Retriever предоставляет агентам минимально достаточный, трассируемый и релевантный набор фрагментов статьи и связанных артефактов. Компонент не формирует факты самостоятельно; его задача — поиск и ранжирование фактов.

## 2. Входы и выходы

### Вход

- `run_id`
- `query_text`
- `query_type` (`method|dataset|metric|hyperparameter|implementation|experiment`)
- `top_k`
- `revision_id`

### Выход

- `EvidencePack`
  - `items[]`
    - `chunk_id`
    - `source_ref`
    - `score_sparse`
    - `score_dense`
    - `score_rerank`
    - `section_path`
    - `text`
  - `empty_result_reason` при пустом результате

## 3. Источники

- текст статьи;
- приложения;
- заголовки таблиц и рисунков;
- текстовые внешние артефакты, явно включённые пользователем в run.

## 4. Индексация

### 4.1 Чанкование

Правила:

- chunk формируется с учётом границ секций;
- сохраняются `document_id`, `section_path`, `page`, `offset_start`, `offset_end`;
- chunk не должен разрывать таблицу или формально связанную подпись;
- размер чанка фиксируется policy-конфигурацией.

### 4.2 Индексы

- sparse index для keyword/BM25-поиска;
- dense index для semantic retrieval;
- metadata filter по секциям и типу источника.

## 5. Алгоритм поиска

1. Выполнить sparse retrieval.
2. Выполнить dense retrieval.
3. Объединить кандидаты по `chunk_id`.
4. Выполнить reranking.
5. Вернуть top-k с provenance.

## 6. Ограничения

- chunks без provenance запрещены;
- при недоступности dense retriever допускается sparse-only режим;
- если reranker недоступен, допускается возврат результата после fusion, но факт деградации логируется;
- retriever не должен сам заполнять отсутствующие сведения.

## 7. Надёжность

- индексация идемпотентна для одинакового `revision_id`;
- перестроение индекса создаёт новую ревизию, не перетирая активную;
- ошибки индексации переводят компонент в `degraded_readonly` для предыдущей ревизии;
- latency и empty-hit rate подлежат мониторингу.

## 8. Ошибки

- `INDEX_NOT_FOUND`
- `INDEX_BUILD_FAILED`
- `EMPTY_RESULT`
- `RERANKER_TIMEOUT`
- `REVISION_MISMATCH`

## 9. Метрики

- p95 latency retrieval;
- empty-hit rate;
- доля evidence-pack с корректным provenance;
- reranker timeout rate;
- index build duration;
- index revision switch failures.
