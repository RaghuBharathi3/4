# 28. Risk Assessment

## Introduction

This document identifies, analyzes, and outlines mitigation strategies for technical, operational, and AI-specific risks associated with the **Generative AI-Powered Cloud Security Assistant**.

---

## Risk Register

```
+---------------------------------------------------------------------------------+
|                                  Risk Matrix                                    |
|                                                                                 |
|  +--------------------+   +--------------------+   +-------------------------+  |
|  |     High Risk      |   |    Medium Risk     |   |        Low Risk         |  |
|  | - LLM Hallucinations|  | - Prompt Injection |   | - Multi-Cloud schema    |  |
|  | - Cost Overruns    |   | - API Rate Limits  |   |   formatting drifts     |  |
|  +--------------------+   +--------------------+   +-------------------------+  |
+---------------------------------------------------------------------------------+
```

---

## Detailed Risk Assessments

### 1. LLM Hallucinations in Remediation Scripts
* **Description**: The LLM generates syntactically invalid commands or, in worst-case scenarios, destructive commands that could cause downtime or security vulnerabilities.
* **Likelihood**: High (due to the creative nature of LLMs).
* **Impact**: Critical (can lead to broken production services or security gaps).
* **Mitigation Strategies**:
  1. Set temperature to 0.0 to enforce deterministic completions.
  2. Implement an outbound validation linter to run structural syntactical validations on code blocks.
  3. Enforce a **Human-in-the-Loop** model. The assistant only suggests commands; it cannot execute them directly without administrator review.

### 2. Prompt Injection Attacks
* **Description**: Malicious users input text that overrides system instructions, tricking the assistant into exposing data or generating malicious commands.
* **Likelihood**: Medium.
* **Impact**: High (can compromise the host system's integrity or database access).
* **Mitigation Strategies**:
  1. Separate system instructions from user inputs in API requests using structured API keys.
  2. Pre-scan queries for injection indicators.
  3. Validate LLM outputs against strict blocklists before displaying them.

### 3. PII & Credential Leakage to Public APIs
* **Description**: Log payloads containing passwords, database strings, or access keys are transmitted to third-party APIs (e.g., OpenAI, Anthropic), violating compliance standards.
* **Likelihood**: Medium.
* **Impact**: Critical (leads to data compliance failures and potential cloud compromises).
* **Mitigation Strategies**:
  1. Run a deterministic sanitization pipeline (regex + NER) before sending log payloads to external APIs.
  2. For sensitive deployments, run a local open-weight model (e.g., Llama 3 70B via vLLM) in an isolated network environment.

### 4. API Cost Overruns
* **Description**: High security event volumes trigger millions of API queries, causing unexpected costs.
* **Likelihood**: High (especially during incident bursts).
* **Impact**: High (can exceed project budgets).
* **Mitigation Strategies**:
  1. Filter incoming logs before sending them to the AI, triaging only verified security alerts.
  2. Use a tiered model routing strategy, sending low-severity alerts to lower-cost models (Gemini 1.5 Flash).
  3. Cache responses for identical alerts.

### 5. API Rate Limiting (HTTP 429 Errors)
* **Description**: The remote LLM provider throttles the assistant's API calls during a security incident.
* **Likelihood**: Medium.
* **Impact**: High (can delay threat triage when it's needed most).
* **Mitigation Strategies**:
  1. Implement exponential backoff and retry logic in backend services.
  2. Set up local rule-based fallback templates to provide basic remediation advice if external APIs are down.
  3. Use local models (e.g., Llama 3 8B) for basic triage during connection failures.
