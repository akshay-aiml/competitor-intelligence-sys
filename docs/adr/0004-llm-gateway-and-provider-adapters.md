# ADR 0004: LLM Gateway Abstraction & Provider Port-Adapter Pattern

## Status
Accepted

## Context
Our primary inference engine is Groq, chosen for its ultra-high token throughput and low latency on open-weight models (Llama-3). However, tightly coupling agent code and prompts directly to the `groq` SDK introduces vendor lock-in risks:
- Inability to seamlessly failover to alternative providers (e.g., Anthropic, OpenAI, or local vLLM) during Groq outages or 429 rate limit spikes.
- Difficulty in mocking LLM interactions cleanly in fast, offline unit test suites.
- Fragmentation of retry, backoff, and token tracking logic across multiple agent nodes.

## Decision
We introduce an **LLM Gateway** abstraction under `src/llm_gateway/`:
- **Port**: A provider interface (`BaseLLMProvider`) declaring `generate_text` and `generate_structured`.
- **Adapters**: Concrete implementations, including `GroqProviderAdapter` and `MockLLMProvider`.
- **Factory**: `LLMFactory` which dynamically resolves the configured provider adapter and handles exponential backoff retries.
- Agent nodes and orchestration workflows are strictly forbidden from importing vendor SDKs; they interact exclusively with the LLM Gateway port.

## Consequences
### Positive
- Complete decoupling of AI business logic from underlying inference providers.
- Painless testing with fast offline mock providers.
- Centralized point for rate-limit handling, telemetry, and fallback routing.
### Negative
- Slight abstraction layer to maintain over raw vendor SDKs.
