# Operational Guide: Telemetry & Monitoring

## 1. Metrics & Instrumentation
The system exposes telemetry via a Prometheus-compatible endpoint at `/metrics`.

### Core Application Metrics
| Metric Name | Type | Description |
| :--- | :--- | :--- |
| `http_requests_total` | Counter | Inbound API requests partitioned by route, method, and status code |
| `http_request_duration_seconds` | Histogram | Request latency distribution |
| `agent_workflow_duration_seconds` | Histogram | End-to-end multi-agent execution duration |
| `agent_node_executions_total` | Counter | Executions of individual graph nodes (crawl, analyze, synthesize) |
| `llm_gateway_requests_total` | Counter | Requests to LLM providers (provider, model, status) |
| `llm_gateway_token_usage_total` | Counter | Tokens consumed partitioned by provider, model, and direction (prompt/completion) |
| `crawling_requests_total` | Counter | Web crawls performed (status, cached_flag) |

---

## 2. Health Probes
FastAPI provides standardized health endpoints for container orchestrators (Kubernetes / Docker Compose):

- **`/health/live` (Liveness)**:
  - Returns `200 OK` if the process is responsive.
- **`/health/ready` (Readiness)**:
  - Verifies connectivity to PostgreSQL (`SELECT 1`) and Redis (`PING`).
  - Returns `200 OK` if all dependencies are reachable; returns `503 Service Unavailable` if an essential dependency is disconnected.

---

## 3. Alerting Thresholds
- **High Error Rate**: Alert when 5xx HTTP responses exceed 2% over a 5-minute sliding window.
- **Agent Execution Drift**: Alert when agent workflow duration p95 exceeds 60 seconds.
- **LLM Provider Degradation**: Alert when Groq returns HTTP 429 or 5xx for more than 3 consecutive retries.
- **Queue / Checkpoint Lag**: Alert when Redis memory utilization exceeds 80%.
