---
trigger: glob
globs: "**/tests/**/*.py, **/*test*.py"
---

# Rule: Testing & Quality Standards

## 1. Testing Philosophy & Progressive Escalation
- **Fast Feedback First**:
  - Unit tests must be hermetic, fast, and runnable offline without external network dependencies.
  - Mock external HTTP services (Tavily, Groq) at the gateway/client interface level using realistic fixture data.
- **Progressive Test Infrastructure**:
  - Start with in-memory SQLite (for relational models) and `MemorySaver` (for LangGraph state) during early development.
  - Escalate to **Testcontainers** for PostgreSQL and Redis integration tests only when verifying concrete database transactions, schema migrations, and real Redis checkpointer behavior.
  - Do not introduce Testcontainers prematurely for trivial unit logic.

## 2. Honest Test Reporting Standards
- **Zero Tolerance for Fake Passes**:
  - Never write empty assertions (`assert True`) or skip tests silently to fake green builds.
  - Never catch all exceptions inside tests to mask failures.
- **Transparent Failure Analysis**:
  - When a test fails during development, report:
    1. Exact failing test name and location.
    2. Stack trace and failure assertion.
    3. Root cause explanation.
    4. Proposed architectural/implementation fix before modifying the code.

## 3. Test Categories & Organization
- `tests/unit/`:
  - Domain entity validation, DTO serialization, LLM Gateway prompt formatting, sanitization logic.
  - Execution time: < 100ms per test.
- `tests/integration/`:
  - Multi-node LangGraph subgraphs, repository database queries, Redis caching round-trips.
- `tests/evaluation/`:
  - Synthetic competitor intelligence evaluation runs using `evaluation.regression`.
  - Asserts model extraction accuracy, hallucination score thresholds, and schema stability.
- **Execution Command**: Always run tests via `uv run pytest -v`.
