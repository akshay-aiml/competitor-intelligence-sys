---
name: debugging-agent-failures
description: >-
  Use this skill when troubleshooting LangGraph infinite loops, Groq rate-limiting (HTTP 429), Tavily search timeouts, or state checkpoint corruption.
---

# Skill: Debugging Agent Failures & Incident Triage

## 1. Overview
This skill provides systematic triage procedures for common multi-agent execution anomalies, external provider rate-limits, and state corruption scenarios.

---

## 2. Common Failure Modes & Resolution Paths

### Failure Mode A: LangGraph Infinite Loops / Cycle Runaways
- **Symptoms**: Agent graph runs indefinitely; step count exceeds threshold; token spend spikes.
- **Root Cause**: Supervisor router fails to reach a terminating condition or oscillation occurs between two worker nodes.
- **Resolution**:
  1. Inspect the graph configuration for `recursion_limit` (ensure maximum steps are capped, default e.g. 15).
  2. Add an explicit state counter `iteration_count: int` incremented at every supervisor decision.
  3. Force transition to `human_gate` or `error_handler` when `iteration_count > max_iterations`.

### Failure Mode B: Groq Rate-Limiting (HTTP 429) & TPM Caps
- **Symptoms**: `groq.RateLimitError` or HTTP 429 received during batch competitor processing.
- **Root Cause**: Token-Per-Minute (TPM) or Request-Per-Minute (RPM) exhaustion on high-throughput runs.
- **Resolution**:
  1. Verify the `LLMFactory` exponential backoff jitter settings (`tenacity` retry with exponential backoff).
  2. Implement an in-memory or Redis token-bucket rate limiter in `GroqProviderAdapter`.
  3. If primary Groq inference remains throttled, trigger gateway fallback adapter.

### Failure Mode C: Tavily Search Timeouts & Scraping Failures
- **Symptoms**: Egress connection timeouts, partial HTML extraction, or empty search responses.
- **Root Cause**: Target competitor site blocking requests or Tavily endpoint transient lag.
- **Resolution**:
  1. Ensure HTTP timeout is explicitly set (`timeout=10.0`).
  2. Catch `CrawlException` at the node level; do not crash the entire workflow graph.
  3. Populate state with `{ "crawl_status": "partial", "errors": [...] }` and allow supervisor to decide whether to continue with cached data.

### Failure Mode D: Checkpoint State Deserialization Errors
- **Symptoms**: State fails to restore from Redis or PostgreSQL when resuming a workflow thread.
- **Root Cause**: Pydantic schema changed between workflow runs or non-serializable objects were injected into state.
- **Resolution**:
  1. Verify that all state fields are JSON-serializable primitives, dicts, or Pydantic models.
  2. Implement schema versioning in state metadata (`schema_version: 1`).
