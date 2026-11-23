# System Architecture Specification

## 1. Architectural Style: Modular Monolith
The system is constructed as a **Modular Monolith**. All capabilities reside within a unified codebase under `src/`, logically organized into discrete bounded contexts with clear ownership, decoupled data stores, and explicit public interfaces.

```mermaid
flowchart TD
    Client([Client / API Consumer]) --> API["API Bounded Context (FastAPI)"]
    
    subgraph Core_Monolith["Modular Monolith (src/)"]
        API --> Orchestration["Orchestration (LangGraph State Machine)"]
        
        Orchestration --> Crawling["Crawling Context (Tavily / SSRF Safe Fetcher)"]
        Orchestration --> Intelligence["Intelligence Context (Entity Extraction)"]
        Orchestration --> Synthesis["Synthesis Context (Executive Briefings)"]
        
        Orchestration --> LLMGateway["LLM Gateway (Provider Abstraction)"]
        Intelligence --> LLMGateway
        Synthesis --> LLMGateway
        
        Orchestration --> Storage["Storage Context (PostgreSQL & Redis)"]
        Intelligence --> Storage
        Synthesis --> Storage
        
        Evaluation["Evaluation Context (Regression & Evals)"] -.-> Orchestration
        Evaluation -.-> LLMGateway
    end
    
    LLMGateway --> Groq[Groq Llama-3 API]
    Crawling --> Tavily[Tavily Search API]
    Storage --> PostgresDB[(PostgreSQL)]
    Storage --> RedisCache[(Redis Cache & Checkpointer)]
    Orchestration --> LangSmith[LangSmith Tracing]
```

---

## 2. Bounded Contexts & Ownership

### 2.1 API Context (`src/api/`)
- **Responsibility**: HTTP routing, request validation, authentication, role-based authorization, SSE streaming for agent progress, RFC 7807 error responses.
- **Dependencies**: Depends on `orchestration`, `intelligence`, `synthesis`. Does not depend on external vendor SDKs directly.

### 2.2 Orchestration Context (`src/orchestration/`)
- **Responsibility**: LangGraph state graph definitions, supervisor routing, workflow state transitions, human-in-the-loop interruption gates, state checkpointing.
- **Dependencies**: Coordinates `crawling`, `intelligence`, `synthesis`, `llm_gateway`, and `storage`.

### 2.3 Crawling Context (`src/crawling/`)
- **Responsibility**: Ingesting raw competitor data from the web. Encapsulates Tavily search queries, raw HTTP retrieval via `httpx`, robots.txt parsing, SSRF URL validation, and HTML sanitization.
- **Dependencies**: Depends on `storage` for caching raw pages. Completely unaware of LLM models or synthesis logic.

### 2.4 Intelligence Context (`src/intelligence/`)
- **Responsibility**: Extracting structured entities (pricing tiers, product capabilities, leadership changes) from sanitized crawl outputs. Comparing extracted signals against historical baselines.
- **Dependencies**: Uses `llm_gateway` for structured inference and `storage` for baseline comparisons.

### 2.5 Synthesis Context (`src/synthesis/`)
- **Responsibility**: Aggregating multi-source competitor signals, evaluating strategic threats, drafting executive briefings, and generating action recommendations.
- **Dependencies**: Uses `llm_gateway` for high-level reasoning and `storage` for historical briefings.

### 2.6 LLM Gateway Context (`src/llm_gateway/`)
- **Responsibility**: Providing a vendor-agnostic interface (`BaseLLMProvider`) for text generation and structured output generation. Implements provider adapters (e.g., `GroqProviderAdapter`, mock adapter), retry policies with exponential backoff, and a dynamic factory.
- **Dependencies**: Standardized on Pydantic and provider SDKs. Isolated from domain logic.

```mermaid
classDiagram
    class BaseLLMProvider {
        <<interface>>
        +generate_text(prompt, system_prompt, **kwargs) LLMResponse
        +generate_structured(schema, prompt, system_prompt, **kwargs) T
    }
    class GroqProviderAdapter {
        -client: Groq
        +generate_text(...) LLMResponse
        +generate_structured(...) T
    }
    class MockLLMProvider {
        +generate_text(...) LLMResponse
        +generate_structured(...) T
    }
    class LLMFactory {
        +get_provider(name: str) BaseLLMProvider
    }

    BaseLLMProvider <|-- GroqProviderAdapter
    BaseLLMProvider <|-- MockLLMProvider
    LLMFactory ..> BaseLLMProvider : creates
```

### 2.7 Evaluation Context (`src/evaluation/`)
- **Responsibility**: Offline benchmarking, quality assurance, and regression gating. Contains:
  - `datasets.py`: Curation of gold-standard and synthetic competitor intelligence samples.
  - `evaluators.py`: Algorithmic and model-graded evaluators measuring extraction precision, recall, factuality, and hallucination.
  - `metrics.py`: Aggregate scoring (F1, hallucination penalty, cost and token tracking).
  - `regression.py`: Test runner that executes evaluation suites in CI/CD before deployments.

### 2.8 Storage Context (`src/storage/`)
- **Responsibility**: Database connectivity, migrations, repository abstractions, caching, and state checkpointing.
  - **PostgreSQL**: Stores structured entities, historical competitor snapshots, reports, and audit logs.
  - **Redis**: Stores ephemeral crawl cache, token rate-limiting buckets, and LangGraph workflow checkpoints.

---

## 3. Data Flow & Request Lifecycle

```mermaid
sequenceDiagram
    autonumber
    actor User as Operator / Client
    participant API as FastAPI Router
    participant Orch as LangGraph Supervisor
    participant Crawl as Crawl Service (Tavily)
    participant Intel as Intelligence Service
    participant Gate as LLM Gateway (Groq)
    participant Store as Storage (Postgres/Redis)

    User->>API: POST /api/v1/intelligence/analyze {competitor_id: "acme"}
    API->>Orch: start_workflow(competitor_id="acme")
    Orch->>Crawl: fetch_competitor_data(target_urls)
    Crawl->>Crawl: Validate SSRF & Sanitize HTML
    Crawl-->>Orch: return CrawlResult
    Orch->>Intel: extract_signals(CrawlResult)
    Intel->>Gate: generate_structured(schema=PricingSignals)
    Gate-->>Intel: validated PricingSignals DTO
    Intel-->>Orch: return ExtractedSignals
    Orch->>Store: persist_signals(competitor_id, signals)
    Orch-->>API: Stream status / Return WorkflowRunId
    API-->>User: HTTP 202 Accepted {run_id: "run-123"}
```
