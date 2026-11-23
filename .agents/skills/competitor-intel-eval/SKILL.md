---
name: competitor-intel-eval
description: >-
  Use this skill when developing, running, or analyzing evaluation benchmarks for competitor intelligence extraction, factuality, hallucination rates, and synthesis regression.
---

# Skill: Competitor Intelligence Evaluation & Regression

## 1. Overview
This skill defines how to execute systematic evaluations against competitor intelligence agents using the `src/evaluation/` module and LangSmith.

---

## 2. Evaluation Architecture

The evaluation pipeline is structured into four core components under `src/evaluation/`:
- `datasets.py`: Ground-truth datasets containing synthetic and curated real-world competitor web samples with known facts (pricing changes, executive movements, product spec launches).
- `evaluators.py`: Custom evaluators measuring:
  - **Entity Precision & Recall**: Did the agent correctly extract pricing tiers and product features?
  - **Factuality & Hallucination**: Does the generated synthesis contain claims unsupported by raw crawl inputs?
  - **Schema Stability**: Did the output strictly adhere to the expected Pydantic DTO schema?
- `metrics.py`: Aggregation algorithms calculating F1 score, hallucination penalty, latency percentiles, and token cost per run.
- `regression.py`: CI/CD regression harness that runs full or slice evaluation suites to prevent performance regressions upon prompt or model changes.

---

## 3. Evaluation Execution Protocol

### Step 1: Prepare Ground Truth Dataset
Ensure target test cases are registered in `datasets.py` or synchronized with LangSmith:
```python
from src.evaluation.datasets import get_competitor_eval_dataset

dataset = get_competitor_eval_dataset(name="pricing_change_v1")
```

### Step 2: Run Evaluation Harness
Run the evaluation suite locally or against LangSmith:
```bash
uv run pytest tests/evaluation/test_regression.py -v
```

### Step 3: Inspect Metrics & Regression Thresholds
Evaluate against predefined quality bars:
- **Factuality Score**: $\ge 0.95$ (less than 5% unsupported assertions).
- **Extraction Precision**: $\ge 0.90$.
- **Schema Compliance**: $1.0$ (zero schema validation failures).

### Step 4: LangSmith Web Verification
1. Inspect trace runs in the LangSmith project `competitor-intel-eval`.
2. Review failed examples to identify whether failures stem from scraping distortion, prompt ambiguity, or model hallucination.
3. Update ground truth or prompt templates accordingly.
