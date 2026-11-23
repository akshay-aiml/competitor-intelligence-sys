# Technology Stack Matrix & Tooling Rationale

## 1. Technology Matrix

| Layer / Capability | Technology | Version / Spec | Primary Responsibility |
| :--- | :--- | :--- | :--- |
| **Language** | Python | `3.12+` | Core programming language across all modules |
| **Package Management** | UV | `latest` | High-performance virtualenv and dependency resolution |
| **Web Framework** | FastAPI | `^0.115.0` | Async REST API, SSE streaming, OpenAPI documentation |
| **Validation & Schemas** | Pydantic | `^2.9.0` | Data validation, settings management, structured LLM outputs |
| **Agent Orchestration** | LangGraph | `^0.2.45` | Deterministic state machine and multi-agent coordination |
| **AI Framework** | LangChain Core | `^0.3.0` | Runnables, standard prompt/message types |
| **Primary LLM Inference** | Groq | Cloud API | High-throughput, sub-second inference (Llama-3 models) |
| **Web Search & Ingestion**| Tavily | Cloud API | Search queries, web scraping with content extraction |
| **Observability & Evals** | LangSmith | Cloud API / SDK | Distributed agent tracing, run metadata, evaluation datasets |
| **Relational Database** | PostgreSQL | `16+` | Durable persistence of entities, reports, and audit logs |
| **Async DB Driver** | asyncpg / SQLAlchemy | `^2.0` (async) | Asynchronous relational database abstraction |
| **Cache & Checkpointing**| Redis | `7+` | Egress cache, rate limits, LangGraph state checkpoints |
| **Code Hygiene** | Ruff | `latest` | Fast linting and opinionated code formatting |
| **Testing Harness** | Pytest | `^8.3.0` | Unit, integration, and evaluation test suites |
| **Containerization** | Docker | Engine `26+` | Multi-stage production container builds |

---

## 2. Engineering Rationale for Core Choices

### 2.1 Python 3.12 & UV
- **Why Python 3.12**: Delivers notable runtime performance improvements (specializing adaptive interpreter), modern typing syntax (`type` keyword, generic functions), and mature async event loop performance.
- **Why UV**: Replaces pip, pip-tools, and poetry with an extremely fast Rust-based package resolver. Generates deterministic `uv.lock` files, eliminating dependency drift between local workstations and CI/Docker builds.

### 2.2 LangGraph over Autonomous Agent Loops
- Unconstrained autonomous agent loops (e.g., raw ReAct loops) often drift into non-deterministic execution cycles, incurring excessive token spend and unbounded latency.
- LangGraph models multi-agent systems as explicit cyclic state graphs with deterministic edges, typed state transitions, and first-class Human-in-the-Loop interrupt support.

### 2.3 LLM Gateway Abstraction with Groq
- Groq provides ultra-fast inference speed for open-weight models (Llama-3.1 70B/8B), enabling interactive multi-step agent reasoning without long user wait times.
- Wrapping Groq inside our `llm_gateway` abstraction ensures agents never depend on proprietary SDK quirks and enables seamless fallback to alternative providers during rate limits.

### 2.4 PostgreSQL & Redis Storage Tiering
- **PostgreSQL**: Provides relational integrity, ACID guarantees, and JSONB capabilities for structured competitor intelligence entities and historical trends.
- **Redis**: Provides sub-millisecond key-value storage for ephemeral crawl caching (preventing redundant Tavily queries) and native support for LangGraph state checkpointing.
