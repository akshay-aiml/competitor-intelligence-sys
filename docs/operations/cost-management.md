# Operational Guide: Cost Management & Token Budgets

## 1. Cost Drivers
Operating costs for the Multi-Agent Competitor Intelligence System stem primarily from:
1. **LLM Inference Tokens**: Calls through the LLM Gateway to Groq (or fallback models).
2. **Search & Crawling Queries**: Queries executed against Tavily API.
3. **Database & Cache Storage**: PostgreSQL relational storage and Redis RAM utilization.

---

## 2. Mitigation Strategies & Quotas

### 2.1 Aggressive Ingestion Caching (Redis)
- Scraped raw competitor HTML pages are hashed and cached in Redis with a configurable Time-To-Live (TTL, default: 24 hours).
- Subsequent analysis or evaluation runs for the same target URL within the TTL read from Redis cache, consuming **0 Tavily credits**.

### 2.2 Token-Conscious Prompt Engineering
- Use specialized, compact extraction prompts rather than dumping massive raw HTML into the context window.
- Pre-filter HTML with the `crawling` sanitization pipeline to eliminate boilerplate (navbars, footers, script tags) before tokenization.

### 2.3 Tiered Model Routing
- Use smaller, high-throughput models (e.g., Llama-3.1 8B on Groq) for high-frequency extraction tasks.
- Reserve larger models (e.g., Llama-3.1 70B on Groq) for high-level multi-source executive synthesis.

---

## 3. Budget Caps & Circuit Breakers
- **Daily Token Ceiling**: Set a hard daily budget cap in `src/config.py`. Once reached, the LLM Gateway transitions non-critical background jobs to queued state.
- **Workflow Step Caps**: Enforce `max_iterations = 15` in all LangGraph supervisors to eliminate accidental runaway loops.
