# ADR 0003: Stateful Multi-Agent Orchestration with LangGraph

## Status
Accepted

## Context
Competitor intelligence synthesis requires coordinating multiple autonomous capabilities: querying search engines, parsing HTML, extracting structured corporate entities, detecting baseline diffs, synthesizing summaries, and requesting human operator review.

Unstructured autonomous multi-agent loops (e.g., unconstrained ReAct loops or multi-agent chat frameworks) exhibit severe failure modes in production:
- Infinite loops between agents debating hypotheses.
- Unbounded token expenditure.
- Inability to pause workflows for human approval and cleanly resume from saved state.
- Difficulties in deterministically reproducing failures.

## Decision
We adopt **LangGraph** as the multi-agent state orchestration framework:
- Multi-agent coordination is modeled as an explicit, cyclic state graph with a designated supervisor router node.
- Workflow state is immutable, strictly-typed (`TypedDict` / Pydantic), and updated via deterministic reducer functions.
- Workflows support first-class state checkpointing (via Redis/PostgreSQL), allowing safe pause/resume for Human-in-the-Loop review gates.

## Consequences
### Positive
- Deterministic, observable state machine control over multi-agent workflows.
- Native integration with LangSmith for granular node-level distributed tracing.
- Resilience against runtime crashes through persistent checkpoints.
### Negative
- Requires upfront schema design and explicit edge/node definitions compared to ad-hoc prompting frameworks.
