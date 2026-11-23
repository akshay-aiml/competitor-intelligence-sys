# LangSmith Observability & Evaluation Strategy

## 1. Overview
LangSmith serves as the unified observability, tracing, and evaluation backbone for the Multi-Agent Competitor Intelligence System. Every agent interaction, tool execution, and LLM call emits structured telemetry to prevent multi-agent drift and ensure empirical performance tracking.

---

## 2. Distributed Tracing Hierarchy

Every invocation of a LangGraph workflow generates a structured trace hierarchy:

```text
Trace Root: CompetitorAnalysisWorkflow [run_id: ...]
├── Tags: ["environment:production", "competitor:acme", "workflow:deep_dive"]
├── Metadata: {"correlation_id": "...", "client_id": "...", "graph_version": "1.0.0"}
│
├── Span: supervisor_node
│   └── Output: {"next_step": "crawling_node"}
│
├── Span: crawling_node
│   ├── Child Span: TavilyClient.search
│   └── Child Span: ContentSanitizer.clean
│
├── Span: supervisor_node
│   └── Output: {"next_step": "analysis_node"}
│
├── Span: analysis_node
│   ├── Child Span: LLMGateway.generate_structured
│   │   ├── Model: "llama-3.1-70b-versatile" (via GroqProviderAdapter)
│   │   ├── Tokens: {prompt: 1420, completion: 310, total: 1730}
│   │   └── Output: Validated PricingSignals DTO
│   └── Output: ExtractedSignals
│
└── Span: synthesis_node
    └── Child Span: LLMGateway.generate_text
```

---

## 3. Evaluation Strategy

### 3.1 Offline Golden Datasets
Managed within `src/evaluation/datasets.py` and synchronized with LangSmith Datasets:
- **Pricing Edge Cases**: Web pages with complex tiers, usage-based billing, enterprise call-out models.
- **Product Launches**: Blog posts and changelogs with implicit feature releases.
- **Noise / Adversarial Scenarios**: Scraped pages containing marketing fluff, cookie banners, and simulated prompt injections.

### 3.2 Evaluation Metrics
- **Extraction Precision & Recall**: Precision/recall of extracted pricing numbers against gold standard ground truth.
- **Hallucination Detection**: Utilizing model-graded evaluation to verify whether claims in the executive summary are grounded in raw crawled source snippets.
- **Latency & Cost**: Tracking p50, p90, p95 latency and token expenditure per workflow run.

### 3.3 CI/CD Regression Gate
Before merging agent prompt or graph logic changes:
```bash
uv run pytest tests/evaluation/test_regression.py
```
A pull request is blocked if:
- Factuality score drops below **0.95**.
- Extraction precision drops below **0.90**.
- Schema validation compliance drops below **1.0**.
