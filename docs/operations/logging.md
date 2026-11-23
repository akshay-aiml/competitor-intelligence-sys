# Operational Guide: Structured Logging & Correlation

## 1. Logging Standard
All logs across the application are output as structured single-line JSON to standard output (`stdout`), formatted for aggregation in tools like Datadog, Grafana Loki, or Google Cloud Logging.

```json
{
  "timestamp": "2026-11-22T03:59:00.123Z",
  "level": "INFO",
  "logger": "src.orchestration.workflow",
  "message": "Node transition completed successfully",
  "correlation_id": "c73a8184-e429-45be-bb37-9759c914bf6f",
  "competitor_id": "acme-corp",
  "node": "analysis_node",
  "duration_ms": 342.1
}
```

---

## 2. Contextual Variables & Propagation
Using Python's `contextvars`, the `correlation_id` is captured in the FastAPI middleware and automatically injected into all downstream logging calls across async tasks.

### Mandatory Log Fields
- `timestamp`: ISO-8601 UTC timestamp.
- `level`: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`.
- `correlation_id`: Unique request or workflow run identifier.
- `logger`: Source Python module name.
- `message`: Descriptive, parameter-free text statement.

---

## 3. Secret & PII Masking
Log formatters must pass messages through a redaction filter before writing to `stdout`:
- Sensitive headers (`Authorization`, `X-API-Key`) are masked as `Bearer ***[REDACTED]***`.
- LLM prompt payloads are truncated in production unless explicitly enabled via `LOG_PROMPTS=true` in development environments.
