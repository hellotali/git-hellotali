## System Architecture – hellotali (MVP)

### Diagram
```mermaid
graph TD
  subgraph "Frontend (Angular 18 PWA)"
    UI["Daily Check-in (text)\nVoice UI (stub)"]
    Charts["Trends view"]
    FEAPI["ApiService (REST)"]
  end

  subgraph "Backend (.NET 8 Web API | Clean Architecture)"
    API["Endpoints: /checkins, /trends, /analysis/voice (stub)"]
    Adapter["IAiAnalyzer → OpenAI (default) | Azure (opt) | Local (fallback)"]
    DB[("PostgreSQL (all environments)")]
    Jobs["Background jobs (trend calc, alerts)"]
    Blob[("Blob storage – audio later")]
    Observability["Logging/metrics"]
  end

  subgraph "AI Services"
    OpenAI["OpenAI GPT models"]
    Azure["Azure Text Analytics (optional)"]
  end

  UI --> FEAPI --> API
  API --> Adapter
  Adapter --> OpenAI
  Adapter --> Azure
  API --> DB
  API --> Jobs
  API --> Observability
```

### Notes
- OpenAI is the default AI provider in MVP; Azure/TextAnalytics and Local analyzer remain available via feature flag.
- PostgreSQL is used across development, staging, and production for parity.
- Voice support is UI + backend stub initially; actual audio processing can be added later.

