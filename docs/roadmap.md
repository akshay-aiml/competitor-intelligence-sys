# Progressive Development Roadmap

## Overview
This roadmap establishes a staged engineering progression. Each phase introduces capabilities only when justified by validated requirements, avoiding premature distributed complexity.

---

## Phase Breakdown

### Phase 0: Engineering Foundation & Tooling Scaffolding (Target: Current)
- [ ] Initialize Python 3.12 project with `pyproject.toml` managed by UV.
- [ ] Configure Ruff for linting and formatting.
- [ ] Establish directory structure matching the modular monolith specification (`src/`, `tests/`).
- [ ] Configure test runner with `pytest`, `pytest-asyncio`, and basic sanity tests.
- [ ] Create base configuration schema using Pydantic Settings (`src/config.py`).

### Phase 1: LLM Gateway & Ingestion Context
- [ ] Define `BaseLLMProvider` interface and data schemas in `src/llm_gateway/`.
- [ ] Implement `GroqProviderAdapter` with retry and backoff logic.
- [ ] Implement `MockLLMProvider` for offline unit testing.
- [ ] Implement `LLMFactory` for provider instantiation.
- [ ] Implement SSRF-safe HTTP client and Tavily integration in `src/crawling/`.
- [ ] Implement HTML sanitization and untrusted content boundaries.

### Phase 2: LangGraph Orchestration & Intelligence Agents
- [ ] Define immutable workflow state schema (`CompetitorWorkflowState`).
- [ ] Implement `crawling_node`, `analysis_node`, and `synthesis_node`.
- [ ] Implement `supervisor_node` for state-driven routing.
- [ ] Integrate LangSmith run metadata and distributed tracing.
- [ ] Build in-memory state checkpointing and test multi-turn graph execution.

### Phase 3: Storage Tiering & Persistence
- [ ] Configure PostgreSQL async engine using SQLAlchemy 2.0 and Alembic migrations.
- [ ] Define relational models for competitors, crawl artifacts, signals, and reports.
- [ ] Implement Redis client for caching scraped pages and managing token rate-limiting.
- [ ] Upgrade LangGraph checkpointer from in-memory to Redis/PostgreSQL.
- [ ] Introduce Testcontainers for PostgreSQL and Redis integration tests.

### Phase 4: API Presentation & Security Lifecycle
- [ ] Build FastAPI application with versioned routers (`/api/v1/`).
- [ ] Implement API key and Bearer token authentication middleware with RBAC.
- [ ] Implement Server-Sent Events (SSE) endpoint to stream real-time agent workflow progress.
- [ ] Implement Human-in-the-Loop review endpoints (`/workflows/{id}/approve`, `/workflows/{id}/reject`).
- [ ] Standardize RFC 7807 problem details error handling.

### Phase 5: Evaluation & Production Hardening
- [ ] Build the evaluation module (`src/evaluation/`):
  - Curate ground-truth dataset in `datasets.py`.
  - Implement factuality and precision evaluators in `evaluators.py`.
  - Implement quantitative scoring in `metrics.py`.
  - Implement CI regression test runner in `regression.py`.
- [ ] Author production multi-stage `Dockerfile` and `docker-compose.yml`.
- [ ] Execute load testing and verify cost management alarms.
