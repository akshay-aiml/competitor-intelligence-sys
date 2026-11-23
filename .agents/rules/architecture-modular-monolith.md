---
trigger: always_on
---

# Rule: Modular Monolith Architecture & Bounded Contexts

## 1. Core Principle
This system is organized as a **Modular Monolith**. Code is partitioned into distinct bounded contexts within `src/`. Each module encapsulates its internal domain logic, data structures, and private persistence details, exposing only a strictly defined public interface.

## 2. Bounded Context Taxonomy & Ownership
- **`src/crawling/`**:
  - *Ownership*: Web ingestion, HTML parsing, Tavily client integration, rate-limiting, robots.txt compliance.
  - *Public API*: `crawling.service.CrawlService`, `crawling.schemas.CrawlResult`.
- **`src/intelligence/`**:
  - *Ownership*: Entity extraction, competitor profiling, pricing/feature diffing, signal extraction.
  - *Public API*: `intelligence.service.IntelligenceService`, `intelligence.schemas.CompetitorProfile`.
- **`src/synthesis/`**:
  - *Ownership*: Multi-source report generation, threat level evaluation, executive summaries.
  - *Public API*: `synthesis.service.SynthesisService`, `synthesis.schemas.ExecutiveBriefing`.
- **`src/orchestration/`**:
  - *Ownership*: LangGraph state machines, multi-agent coordination, supervisor nodes, checkpointing.
  - *Public API*: `orchestration.workflow.CompetitorWorkflow`, `orchestration.state.WorkflowState`.
- **`src/llm_gateway/`**:
  - *Ownership*: Provider-agnostic LLM interface (`BaseLLMProvider`), provider adapters (Groq, mock), factory.
  - *Public API*: `llm_gateway.factory.LLMFactory`, `llm_gateway.base.BaseLLMProvider`.
- **`src/evaluation/`**:
  - *Ownership*: Offline datasets, custom evaluators, quantitative metrics, regression test harness.
  - *Public API*: `evaluation.regression.RegressionHarness`, `evaluation.evaluators.FactualityEvaluator`.
- **`src/storage/`**:
  - *Ownership*: Database engines, PostgreSQL models/migrations, Redis caching clients, repositories.
  - *Public API*: `storage.postgres.engine`, `storage.redis.client`, domain repository interfaces.
- **`src/api/`**:
  - *Ownership*: FastAPI controllers, route definitions, auth/RBAC middleware, request lifecycle.
  - *Public API*: FastAPI application entrypoint.

## 3. Dependency Rules & Prohibitions
1. **Unidirectional Dependency Flow**:
   - `api` may depend on `orchestration`, `intelligence`, `synthesis`, `crawling`, `evaluation`.
   - `orchestration` coordinates domain modules (`crawling`, `intelligence`, `synthesis`) and calls `llm_gateway`.
   - Domain modules (`crawling`, `intelligence`, `synthesis`) **never** import `api` or `orchestration`.
   - Domain modules depend on abstractions provided by `storage` or `llm_gateway`, never on concrete presentation code.
2. **No Direct Cross-Module Database Queries**:
   - The `intelligence` module cannot query `crawling` private tables directly. If it needs crawl data, it calls the `CrawlService` or receives a `CrawlResult` DTO.
3. **No Circular Imports**:
   - Circular imports between modules are strictly prohibited. Decouple through domain events, DTOs, or shared interfaces.
