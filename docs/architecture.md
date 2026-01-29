# System Architecture – hellotali

## Overview

hellotali uses a **Vertical Slice Architecture** where features are organized as self-contained slices rather than horizontal layers. Each slice contains its own endpoints, handlers, models, and tests.

---

## High-Level Architecture

```mermaid
graph TB
    subgraph Client["Client Layer"]
        Web["Web Browser (PWA)"]
        Mobile["Mobile (Capacitor)"]
    end

    subgraph Frontend["Frontend - Angular 21"]
        FE_CheckIn["CheckIn Feature"]
        FE_Trends["Trends Feature"]
        FE_Billing["Billing Feature"]
        FE_Core["Core Services"]
        FE_Shared["Shared Components"]
    end

    subgraph Backend["Backend - .NET 10"]
        API["API Host"]
        subgraph Features["Features (Vertical Slices)"]
            BE_CheckIn["CheckIn"]
            BE_Trends["Trends"]
            BE_Analysis["Analysis"]
            BE_Billing["Billing"]
        end
        subgraph Shared["Shared Kernel"]
            Domain["Domain"]
            Contracts["Contracts"]
        end
        subgraph Infra["Infrastructure"]
            DbCtx["DbContext"]
            Repos["Repositories"]
            AI["AI Adapters"]
            Usage["Usage Tracker"]
        end
    end

    subgraph External["External Services"]
        OpenAI["OpenAI API"]
        Azure["Azure Text Analytics"]
        DB[("PostgreSQL")]
    end

    Client --> Frontend
    Frontend -->|REST| API
    API --> Features
    Features --> Shared
    Features --> Infra
    Infra --> External
```

---

## Backend Layers

```mermaid
graph LR
    subgraph Api["HelloTali.Api"]
        Host["Program.cs"]
        MW["Middleware"]
    end

    subgraph Features["HelloTali.Features"]
        F1["CheckIn Slice"]
        F2["Trends Slice"]
        F3["Billing Slice"]
    end

    subgraph Domain["HelloTali.Domain"]
        Entities["Entities"]
        ValueObj["Value Objects"]
        DomainSvc["Domain Services"]
    end

    subgraph Contracts["HelloTali.Contracts"]
        IAi["IAiAnalyzer"]
        IRepo["IRepository"]
        ISvc["IServices"]
    end

    subgraph Infra["HelloTali.Infrastructure"]
        EF["EF Core"]
        Adapters["AI Adapters"]
        Services["Services"]
    end

    Api --> Features
    Features --> Domain
    Features --> Contracts
    Contracts -.->|implemented by| Infra
    Infra --> EF
```

---

## Feature Slice Structure

Each feature is self-contained:

```mermaid
graph TD
    subgraph CheckInSlice["CheckIn Feature Slice"]
        Endpoints["CheckInEndpoints.cs"]
        
        subgraph Commands["Commands"]
            CreateCmd["CreateCheckInCommand"]
            CreateHandler["CreateCheckInHandler"]
        end
        
        subgraph Queries["Queries"]
            GetQuery["GetCheckInQuery"]
            GetHandler["GetCheckInHandler"]
        end
        
        Models["DTOs / Models"]
    end

    Endpoints --> Commands
    Endpoints --> Queries
    Commands --> Models
    Queries --> Models
```

---

## AI Adapter Strategy

```mermaid
graph LR
    subgraph Handler["Feature Handler"]
        Analyze["AnalyzeMoodHandler"]
    end

    subgraph Contract["Contract"]
        Interface["IAiAnalyzer"]
    end

    subgraph Implementations["Infrastructure"]
        OpenAI["OpenAiAnalyzer"]
        Azure["AzureTextAnalyzer"]
        Local["LocalRuleBasedAnalyzer"]
    end

    Handler --> Interface
    Interface -.-> OpenAI
    Interface -.-> Azure
    Interface -.-> Local
```

Switching via `AI_PROVIDER` environment variable:
- `openai` (default) → OpenAiAnalyzer
- `azure` → AzureTextAnalyzer
- `local` → LocalRuleBasedAnalyzer (fallback)

---

## Data Flow: Check-in Request

```mermaid
sequenceDiagram
    participant Client
    participant API as API Host
    participant Handler as CreateCheckInHandler
    participant Usage as UsageTracker
    participant AI as IAiAnalyzer
    participant Repo as ICheckInRepository
    participant DB as PostgreSQL

    Client->>API: POST /api/checkins
    API->>Handler: CreateCheckInCommand
    Handler->>Usage: CheckUsageStatus()
    
    alt Status = Active
        Handler->>AI: AnalyzeAsync(text)
        AI-->>Handler: AnalysisResult + tokensUsed
        Handler->>Usage: RecordUsage(tokens)
        Handler->>Repo: SaveAsync(checkIn)
        Repo->>DB: INSERT
        Handler-->>API: CheckInDto
        API-->>Client: 201 Created
    else Status = Suspended
        Handler->>AI: LocalAnalyzer.Analyze(text)
        AI-->>Handler: AnalysisResult (local)
        Handler->>Repo: SaveAsync(checkIn)
        Repo->>DB: INSERT
        Handler-->>API: CheckInDto + warning
        API-->>Client: 201 Created (degraded)
    end
```

---

## Usage Monitoring Flow

```mermaid
flowchart TD
    Request["AI Request"]
    Check["Check Usage Status"]
    
    subgraph StatusCheck["Status Check"]
        Active["Status: Active"]
        Warning["Status: Warning"]
        Suspended["Status: Suspended"]
    end
    
    subgraph Actions["Actions"]
        Proceed["Use OpenAI"]
        Warn["Use OpenAI + Notify"]
        Fallback["Use Local AI"]
    end
    
    Update["Update TokensUsed"]
    Threshold["Check Thresholds"]
    
    Request --> Check
    Check --> Active --> Proceed
    Check --> Warning --> Warn
    Check --> Suspended --> Fallback
    
    Proceed --> Update
    Warn --> Update
    Fallback --> Update
    
    Update --> Threshold
    Threshold -->|">80%"| Warning
    Threshold -->|">100%"| Suspended
```

---

## Frontend Architecture

```mermaid
graph TB
    subgraph App["Angular App"]
        Routes["app.routes.ts"]
        Config["app.config.ts"]
    end

    subgraph Features["Features"]
        CheckIn["check-in/"]
        Trends["trends/"]
        Billing["billing/"]
    end

    subgraph Core["Core"]
        ApiSvc["ApiService"]
        StorageSvc["StorageService"]
        Interceptors["Interceptors"]
    end

    subgraph Shared["Shared"]
        Components["Components"]
        Pipes["Pipes"]
    end

    App --> Features
    Features --> Core
    Features --> Shared
    Core -->|HTTP| Backend["Backend API"]
```

---

## Deployment Architecture

```mermaid
graph LR
    subgraph Client["Client"]
        Browser["Browser"]
    end

    subgraph CDN["Static Hosting"]
        Angular["Angular PWA"]
    end

    subgraph Azure["Azure"]
        AppService["App Service"]
        PGDB[("PostgreSQL")]
    end

    subgraph External["External"]
        OpenAI["OpenAI API"]
    end

    Browser --> Angular
    Angular --> AppService
    AppService --> PGDB
    AppService --> OpenAI
```

---

## Key Design Decisions

1. **Vertical Slices over Layers** - Features are self-contained for easier maintenance
2. **Contracts for DI** - All dependencies injected via interfaces
3. **Domain Purity** - Business logic has no external dependencies
4. **AI Fallback** - Local analyzer ensures service continuity
5. **Usage Tracking** - Built-in cost control and billing support
