---
trigger: glob
globs: "**/agents/**/*.py, **/graphs/**/*.py, **/llm/**/*.py, **/orchestration/**/*.py"
---

# Rule: AI Engineering, LangGraph & LLM Gateway

## 1. LLM Gateway Decoupling
- **No Direct Provider SDK Imports**: Agents and workflows must **never** import `groq`, `langchain_groq`, or other concrete vendor libraries directly.
- **Provider Port & Adapter Pattern**:
  - All LLM access is mediated by `src.llm_gateway.base.BaseLLMProvider`.
  - Concrete adapters (e.g., `GroqProviderAdapter`) implement standard methods:
    - `async def generate_text(...) -> LLMResponse`
    - `async def generate_structured(schema: type[T], ...) -> T`
  - Access is resolved via `LLMFactory.get_provider(name="default")`.
  - This allows transparent fallback, mocking in test suites, and multi-model routing without agent rewrites.

## 2. LangGraph Architecture & State Hygiene
- **Immutable State Definitions**:
  - Define state using `TypedDict` or Pydantic `BaseModel`.
  - Use explicit reducer functions (e.g., `operator.add` or custom merge reducers) when state keys accumulate messages, crawl results, or extracted entities.
- **Deterministic Node Execution**:
  - Each graph node must be an idempotent, focused async function receiving the current state and returning a delta dict.
  - Never mutate state objects in-place.
- **Supervisor & Router Topologies**:
  - The supervisor node determines the next active agent (e.g., `crawling_agent` -> `analysis_agent` -> `synthesis_agent` -> `human_review` -> `END`).
  - Implement edge conditional routing with strict schema validation.
- **Checkpointer Configuration**:
  - Multi-turn interactions and state persistence must use LangGraph checkpointers (e.g., Redis or PostgreSQL checkpointers in production, `MemorySaver` in fast unit tests).

## 3. LangSmith Observability Integration
- **Run Metadata & Tags**:
  - Every graph invocation must specify run configuration with tags: `[competitor_id, workflow_type, environment]`.
  - Inject correlation IDs from FastAPI request context into LangSmith metadata.
- **Trace Hygiene**:
  - Ensure parent-child run relationships are preserved across nested subgraphs and tool executions.
  - Redact sensitive customer parameters before passing state to trace logs.
