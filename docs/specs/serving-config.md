# Serving / Config Specification

## 1. Назначение

Спецификация задаёт требования к развертыванию, конфигурации, секретам, версиям моделей и управлению ресурсами.

## 2. Конфигурационные домены

### 2.1 Runtime config

- worker pool sizes;
- queue thresholds;
- stage timeouts;
- sandbox limits;
- feature flags для degraded mode.

### 2.2 Model config

- primary/secondary provider;
- model mapping по `model_class`;
- max context;
- output token reserve;
- retry/fallback thresholds.

### 2.3 Budget config

- soft/hard token limits;
- cost caps per run;
- concurrency caps;
- retry caps per dependency.

## 3. Секреты

- секреты хранятся вне репозитория;
- инъекция секретов выполняется через runtime secret store;
- секреты не должны попадать в application logs, traces и package artifacts.

## 4. Версионирование

- версия orchestrator logic;
- версия schemas;
- версия retrieval pipeline;
- версия prompts/templates;
- версия моделей и provider configs.

Все версии должны входить в `RunRecord` для воспроизводимости.

## 5. Health endpoints

Минимальный набор:

- `liveness` — процесс жив;
- `readiness` — доступны обязательные зависимости;
- `degraded readiness` — система готова принимать ограниченные runs;
- `provider health` — агрегированная доступность LLM providers.

## 6. Надёжность эксплуатации

- rolling update без потери активных checkpoint;
- graceful shutdown worker с завершением текущего шага или requeue;
- admission control при перегрузке;
- dead-letter queue для невосстановимых job.

## 7. Метрики

- worker utilization;
- queue lag;
- admission rejects;
- graceful shutdown completion rate;
- config drift incidents.
