```mermaid
graph TB
    %% === START ===
    START(["<b>PROJECT START</b><br/>4Ps Program Analysis"])

    %% === SIMPLIFIED PHASES ===
    P0["<b>Phase 0:</b><br/>Define Objectives & Data Sources"]
    P1["<b>Phase 1:</b><br/>Collect & Extract Data"]
    P2["<b>Phase 2:</b><br/>Set Up Infrastructure<br/>(dlt, GitHub, Docker, DBeaver RAW)"]
    P3["<b>Phase 3:</b><br/>Transform Data (dbt)"]
    P4["<b>Phase 4:</b><br/>Validate Data Quality"]
    P5["<b>Phase 5:</b><br/>Model Data<br/>(Schemas, Mart)"]
    P6["<b>Phase 6:</b><br/>Visualize Insights<br/>(Tableau Dashboards)"]
    P7["<b>Phase 7:</b><br/>Deliver Outputs<br/>(Docs & Presentation)"]
    FINAL(["<b>PROJECT COMPLETE</b><br/>4Ps Insights Delivered"])

    %% === FLOW ===
    START --> P0 --> P1 --> P2 --> P3 --> P4 --> P5 --> P6 --> P7 --> FINAL

    %% === NOTES ===
    note right of P2
        GitHub and Docker act as the project backbone
        for version control and environment consistency.
    end note

    %% === STYLES ===
    classDef start fill:#f8f9fa,stroke:#000,stroke-width:2px;
    classDef phase fill:#e3f2fd,stroke:#0d47a1,stroke-width:2px;
    classDef final fill:#d4edda,stroke:#155724,stroke-width:2px;

    class START start;
    class P0,P1,P2,P3,P4,P5,P6,P7 phase;
    class FINAL final;
```
