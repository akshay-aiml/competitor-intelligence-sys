---
trigger: glob
globs: "**/*.py, **/pyproject.toml, **/uv.lock"
---

# Rule: Python 3.12 & UV Standards

## 1. Tooling & Environment Mandates
- **Package Management**: **Strictly use UV**.
  - Add dependencies: `uv add <package>` (or `uv add --dev <package>`)
  - Run commands: `uv run <command>` (e.g., `uv run pytest`, `uv run ruff check .`)
  - Lock dependencies: `uv lock`
  - Sync environment: `uv sync`
  - **Never** invoke raw `pip`, `pip-tools`, or `poetry`.
- **Linter & Formatter**: Use **Ruff**.
  - Format check: `uv run ruff format --check .`
  - Lint check: `uv run ruff check .`

## 2. Python 3.12 Idioms & Typing
- **Type Annotations**:
  - Use modern built-in generics: `list[str]`, `dict[str, Any]`, `tuple[int, ...]`, `set[UUID]` (never import `List`, `Dict`, `Tuple` from `typing`).
  - Use union syntax: `str | None` instead of `Optional[str]`.
  - Prefer the `type` statement for type aliases: `type NodeID = str`.
- **Data Validation & Schemas**:
  - Use **Pydantic v2** (`BaseModel`, `Field`, `ConfigDict`) for boundary validation, DTOs, and LLM structured outputs.
  - For immutable domain records inside internal services, consider standard `dataclasses(frozen=True)` or Pydantic with `frozen=True`.
- **Async-First Execution**:
  - All I/O operations (FastAPI endpoints, HTTP requests via `httpx`, database calls via `asyncpg`/SQLAlchemy async, Redis calls, LangGraph node execution) must be asynchronous (`async def` / `await`).
  - Never run blocking I/O calls inside async coroutines. If synchronous third-party code must run, wrap it via `asyncio.to_thread`.
- **Pattern Matching & Control Flow**:
  - Leverage `match ... case` for state machine transitions, message dispatching, and agent action routing.
