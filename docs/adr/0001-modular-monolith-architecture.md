# ADR 0001: Adoption of Modular Monolith Architecture

## Status
Accepted

## Context
We are designing a production-oriented Multi-Agent Competitor Intelligence System. The system comprises ingestion/crawling, structured entity extraction, multi-agent workflow orchestration, multi-source executive synthesis, offline evaluation, and an external REST/SSE API.

A common industry temptation in multi-agent architectures is to immediately partition each domain capability into independent microservices (e.g., Crawl Service, LLM Extraction Service, Synthesis Service). However, distributed microservices at this stage introduce substantial overhead:
- Network latency across inter-service RPC calls.
- Distributed transaction and distributed tracing complexity.
- High operational overhead (managing multiple CI/CD pipelines, container deployments, and service discovery).
- Slower developer iteration during early architectural stabilization.

## Decision
We adopt a **Modular Monolith** architecture:
- All bounded contexts live within a single repository and deployment unit under `src/`.
- Bounded contexts (`crawling`, `intelligence`, `synthesis`, `orchestration`, `api`, `llm_gateway`, `evaluation`, `storage`) have strict ownership, isolated internal domain logic, and explicitly exported public interfaces (services and DTOs).
- Direct cross-module private imports and cross-boundary database table queries are prohibited.

## Consequences
### Positive
- Unified, fast development workflow with a single virtual environment and atomic test suite.
- Zero network hop latency between domain modules.
- Clean bounded contexts preserve the option to cleanly carve out any module into an independent microservice later if traffic or team scale necessitates it.
### Negative
- Requires strict adherence to boundary discipline to prevent architecture erosion into an unmaintainable "spaghetti" monolith.
