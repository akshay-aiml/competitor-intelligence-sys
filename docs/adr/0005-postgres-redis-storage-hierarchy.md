# ADR 0005: Dual-Tier Storage Architecture (PostgreSQL & Redis)

## Status
Accepted

## Context
The Competitor Intelligence System handles diverse data persistence requirements:
1. **Durable Relational Data**: Structured competitor profiles, historical pricing diffs, executive intelligence reports, audit trails, and user permissions.
2. **High-Speed Ephemeral Data**: Scraped raw web page caches, rate-limiting token buckets, and real-time LangGraph workflow state checkpoints.

Relying exclusively on a relational database for high-frequency crawl caching and fine-grained agent checkpointing creates database connection contention and I/O pressure. Conversely, relying solely on an in-memory or document store compromises relational query integrity and auditability.

## Decision
We adopt a **Dual-Tier Storage Architecture**:
- **PostgreSQL 16 (Tier 1 - Durable Relational)**: Managed via SQLAlchemy 2.0 (async) and Alembic migrations. Stores structured domain models, historical time-series signals, generated executive reports, and RBAC policies.
- **Redis 7 (Tier 2 - Ephemeral Cache & State Machine Checkpointer)**: Stores raw scraped web responses with TTL to eliminate duplicate Tavily search queries, maintains token-bucket rate limits, and provides checkpoint storage for LangGraph thread persistence.

## Consequences
### Positive
- Optimal alignment between data access patterns and storage engine capabilities.
- Sub-millisecond state transitions and crawl cache lookups via Redis.
- ACID guarantees and structured SQL querying for executive competitor reports.
### Negative
- Requires operating and monitoring two distinct database engines in production environments.
