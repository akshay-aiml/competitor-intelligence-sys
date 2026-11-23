---
name: code-review-verification
description: >-
  Use this skill when performing architectural reviews, verifying modular boundary compliance, auditing SOLID/DRY principles, or checking type hygiene.
---

# Skill: Architectural Code Review & Quality Verification

## 1. Overview
This skill provides an authoritative checklist and automated verification procedure for ensuring all proposed changes comply with the system's modular monolith principles, typing standards, and security mandates.

---

## 2. Review Rubric

### 1. Modular Monolith & Separation of Concerns
- [ ] Are boundaries respected? Does code in `crawling` avoid importing `intelligence` or `api`?
- [ ] Are cross-module communications passing through DTOs and public services?
- [ ] Are database models private to the `storage` module or domain repository?
- [ ] Does presentation code (`api`) remain strictly focused on HTTP translation, auth, and request lifecycle?

### 2. LLM Gateway & Provider Decoupling
- [ ] Is there **zero** direct import of `groq` or vendor SDKs in agent code?
- [ ] Are LLM invocations routed through `BaseLLMProvider` via `LLMFactory`?
- [ ] Are structured outputs validated using Pydantic schemas?

### 3. AI Security & Defensive Engineering
- [ ] Does all web crawling pass through SSRF validation (private IP blocking, URL scheme restriction)?
- [ ] Is untrusted scraped content sanitized before inclusion in LLM prompts?
- [ ] Are API credentials, bearer tokens, and internal keys properly redacted from logs?
- [ ] Do high-impact actions have Human-in-the-Loop checkpoints?

### 4. Python 3.12 & Typing Standards
- [ ] Are modern built-in generics used (`list[str]`, `dict[str, Any]`, `X | None`)?
- [ ] Does the code pass Ruff linting and formatting without warnings?
- [ ] Are async/await patterns strictly non-blocking?

### 5. Test Honesty & Coverage
- [ ] Are tests testing genuine behavior rather than mocked trivia?
- [ ] Are test failure reports transparent and accurate?
- [ ] Do unit tests run quickly and offline?

---

## 3. Automated Verification Commands
Run all checks in a single validation chain:
```bash
uv run ruff check . && uv run ruff format --check . && uv run pytest -v
```
