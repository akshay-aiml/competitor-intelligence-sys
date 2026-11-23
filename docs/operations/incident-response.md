# Operational Guide: Incident Response & Runbooks

## 1. Incident Severity Definitions
- **SEV-1 (Critical)**: Entire API unavailable, persistent database connection failure, or widespread agent runaway consuming unbounded LLM tokens.
- **SEV-2 (Major)**: Primary LLM provider (Groq) down with fallback degraded; crawling completely blocked due to external API errors.
- **SEV-3 (Minor)**: Single competitor site crawl failure; non-critical telemetry or LangSmith logging degraded.

---

## 2. Runbook: Agent Runaway / Infinite Loop
### Trigger
Prometheus alert `AgentWorkflowDurationSpike` fires or token consumption rate exceeds threshold.
### Action Steps
1. **Identify the Run**: Search logs for active `correlation_id` values associated with prolonged `agent_node_executions_total`.
2. **Halt Execution**: Trigger the circuit-breaker endpoint:
   ```bash
   curl -X POST https://api.competitor-intel.internal/api/v1/admin/workflows/{workflow_id}/abort \
     -H "Authorization: Bearer $ADMIN_TOKEN"
   ```
3. **Verify Recursion Limits**: Ensure `recursion_limit` is enforced in LangGraph config.
4. **Inspect State in Checkpointer**: Dump Redis checkpoint for the offending thread to diagnose the supervisor router state loop.

---

## 3. Runbook: LLM Provider (Groq) Outage / 429 Throttling
### Trigger
Alert `LLMGatewayRateLimitSpike` or `DownstreamProvider5xx`.
### Action Steps
1. **Check Status**: Inspect Groq operational status page.
2. **Switch Provider Adapter**:
   Update environment variable or dynamic config in Redis:
   ```bash
   LLM_DEFAULT_PROVIDER="fallback_openai" # or backup provider configured in LLMFactory
   ```
3. **Flush / Backoff Worker Queues**:
   Temporarily throttle concurrent crawl workflows to reduce Token-Per-Minute consumption.
