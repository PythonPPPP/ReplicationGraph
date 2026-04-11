# Architecture Diagrams

Ниже приведены диаграммы в формате. Они описывают границы системы, контейнеры, внутреннее устройство ядра, execution flow и data flow.

## 1. C4 Context

```mermaid
flowchart LR
    User[Пользователь / исследователь]
    Paper[Научная статья и связанные артефакты]
    System[PoC-система репликации]
    LLM[LLM providers]
    Store[Graph/Artifact/Run storage]
    Obs[Monitoring & Alerting]

    User -->|запрос на прогон| System
    Paper -->|входные данные| System
    System -->|вызовы моделей| LLM
    System -->|чтение/запись состояния| Store
    System -->|метрики, логи, трейсы| Obs
    System -->|репликационный пакет| User
```

## 2. C4 Container

```mermaid
flowchart TB
    subgraph System[PoC-system]
        API[API / Frontend]
        ORCH[Orchestrator]
        Q[Task Queue]
        ING[Ingestion]
        RET[Retriever]
        EXT[Extraction Agent]
        KG[Knowledge Graph Layer]
        PLN[Planner]
        GEN[Code Generation Agent]
        VER[Verification Agent]
        PKG[Packaging]
        GW[LLM Gateway]
        SBX[Sandbox]
        OBS[Observability]
    end

    API --> ORCH
    ORCH --> Q
    Q --> ING
    Q --> RET
    Q --> EXT
    Q --> PLN
    Q --> GEN
    Q --> VER
    ING --> KG
    RET --> EXT
    EXT --> KG
    PLN --> KG
    PLN --> GEN
    GEN --> SBX
    GEN --> KG
    VER --> KG
    VER --> PKG
    GW --> EXT
    GW --> PLN
    GW --> GEN
    GW --> VER
    OBS --- API
    OBS --- ORCH
    OBS --- Q
    OBS --- GW
    OBS --- SBX
```

## 3. C4 Component: ядро оркестрации

```mermaid
flowchart LR
    subgraph OrchestratorCore[Orchestrator Core]
        SM[State Machine]
        SCH[Step Scheduler]
        BUD[Budget Controller]
        RETRY[Retry/Fallback Manager]
        CKP[Checkpoint Manager]
        POL[Policy Guard]
    end

    SM --> SCH
    SCH --> BUD
    SCH --> RETRY
    SCH --> CKP
    SCH --> POL
    RETRY --> SM
    CKP --> SM
    POL --> SM
```

## 4. Workflow / graph diagram

```mermaid
flowchart TD
    A[Start run] --> B[INGEST]
    B --> C{Корпус валиден?}
    C -- нет --> C1[FAILED / PARTIAL_COMPLETE]
    C -- да --> D[EXTRACT]
    D --> E{Schema valid?}
    E -- нет --> E1[Retry / fallback / DEGRADED]
    E1 --> E2{Достигнут лимит?}
    E2 -- да --> C1
    E2 -- нет --> D
    E -- да --> F[NORMALIZE + WRITE KG]
    F --> G[PLAN]
    G --> H{Минимальная полнота фактов достигнута?}
    H -- нет --> H1[PARTIAL_COMPLETE]
    H -- да --> I[GENERATE]
    I --> J[VERIFY]
    J --> K{Consistency pass?}
    K -- нет --> K1[Mark conflicts / unresolved]
    K1 --> L[PACKAGE]
    K -- да --> L[PACKAGE]
    L --> M[COMPLETE or PARTIAL_COMPLETE]
```

## 5. Data flow diagram

```mermaid
flowchart LR
    IN[Input article] --> MAN[Document manifest]
    MAN --> CH[Chunk set]
    CH --> IDX[Hybrid index]
    IDX --> EV[Evidence pack]
    EV --> FACT[Fact candidates]
    FACT --> VAL[Schema + grounding validation]
    VAL --> KG[Knowledge graph revision]
    KG --> PLAN[Plan items]
    PLAN --> ART[Generated artifacts]
    ART --> CHECK[Verification results]
    CHECK --> PACK[Replication package]

    RUN[Run state] --> PLAN
    RUN --> ART
    RUN --> CHECK
    ART --> LOGS[Logs / metrics / traces]
    CHECK --> LOGS
```
