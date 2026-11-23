# ADR 0002: Adoption of UV for Dependency & Environment Management

## Status
Accepted

## Context
Python dependency management has historically suffered from performance bottlenecks and tool fragmentation across `pip`, `virtualenv`, `pip-tools`, `poetry`, and `pipenv`. In a multi-agent AI system with fast iteration cycles and heavy scientific/AI libraries (LangChain, LangGraph, Pydantic, SQLAlchemy), resolving and syncing dependencies across developer environments, CI pipelines, and Docker builds frequently causes dependency drift and excessive build times.

## Decision
We standardize strictly on **UV** (by Astral) as the unified Python package manager and execution runner.
- All dependencies are declared in `pyproject.toml` and locked into `uv.lock`.
- Tool invocations, testing, and linting are executed via `uv run` (e.g., `uv run pytest`, `uv run ruff check`).
- Manual usage of `pip install`, `poetry`, or external virtual environment activations is prohibited.

## Consequences
### Positive
- Sub-second dependency resolution and lightning-fast virtual environment synchronization.
- Deterministic cross-platform lockfile (`uv.lock`) prevents dependency divergence between local and containerized environments.
- Streamlined CI and Docker container builds.
### Negative
- Developers must be familiar with UV CLI commands rather than standard `pip` invocations.
