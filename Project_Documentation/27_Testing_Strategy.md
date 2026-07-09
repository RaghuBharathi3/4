# 27. Testing Strategy

## Introduction

The testing strategy establishes a validation framework to verify that the **Generative AI-Powered Cloud Security Assistant** meets performance, security, and accuracy standards. Because this system combines traditional code paths with non-deterministic Generative AI responses, it requires a mix of traditional testing suites and specialized LLM evaluations.

---

## 1. Traditional Testing Matrix

```
+---------------------------------------------------------------------------------+
|                                 Testing Matrix                                  |
|                                                                                 |
|  +--------------------+   +--------------------+   +-------------------------+  |
|  |     Unit Tests     |   | Integration Tests  |   |      System Tests       |  |
|  | - Parser logic     |   | - Queue buffer     |   | - End-to-End ingestion  |  |
|  | - Sanitizer regex  |   | - DB updates       |   | - WebSocket chat flows  |  |
|  | - Pydantic models  |   | - Vector search    |   | - Report generation     |  |
|  +--------------------+   +--------------------+   +-------------------------+  |
+---------------------------------------------------------------------------------+
```

### Unit Testing
* **Target Components**: `LogSanitizer`, `NormalizedEvent` validator, and RAG document splitters.
* **Methodology**: Isolates classes and run mock datasets to verify parser behaviors.
* **Verification Examples**:
  * Assert that a mock CloudTrail log with an active AWS access key returns a sanitized payload with the key replaced by `<REDACTED_AWS_ACCESS_KEY>`.
  * Verify that the parser flags and rejects logs with malformed JSON syntax.

### Integration Testing
* **Target Components**: Ingress APIs to Redis queue transitions, queue workers to database writes, and orchestrators to vector DB engines.
* **Methodology**: Uses test databases and local caches to verify communications between microservices.
* **Verification Examples**:
  * Send 100 alerts to the ingestion gateway and verify that all 100 are successfully parsed, written to PostgreSQL, and queued for AI analysis.
  * Verify that a query to `VectorDBManager` returns the correct grounding document chunks.

### System Testing (End-to-End)
* **Target Components**: Complete pipeline operations from log ingestion webhooks to browser WebSocket updates.
* **Methodology**: Simulates real-world incidents, tracking log data transitions through the entire system.
* **Verification Examples**:
  * Trigger a simulated AWS GuardDuty alert webhook and verify that the administrator's browser dashboard renders the corresponding warning card with the correct severity level and remediation commands in under 15 seconds.

---

## 2. Performance and Load Testing

### Latency Constraints
* **Ingestion Throughput**: Run load simulators (e.g., Locust, k6) to verify the gateway handles up to **2,000 requests per minute** with sub-500ms response times.
* **Vector Search Latency**: Confirm that index similarity queries maintain a latency under **100ms** at a concurrency level of 100 queries/sec.

### AI Streaming Performance
* **Time-to-First-Token (TTFT)**: Track WebSocket performance to ensure token streaming starts in under **1.5 seconds** from prompt generation.

---

## 3. Security and Vulnerability Auditing

* **Static Application Security Testing (SAST)**: Run tools like `Bandit` (for Python) to search codebases for security vulnerabilities (e.g., hardcoded passwords, unsafe imports).
* **Dependency Scanning**: Use tools like `Snyk` or `Trivy` to check libraries and Docker parent images for vulnerabilities.
* **Adversarial Pen-Testing**: Simulate prompt injection attacks to verify that system prompts resist override attempts.

---

## 4. Specialized AI and LLM Evaluation

Traditional code validation tests if an output matches a static string. With LLMs, we evaluate output distribution quality:

### 1. Grounding and Retrieval Precision (RAG Evaluation)
* **Metric**: Context Relevance and Faithfulness (using frameworks like `Ragas` or `TruLens`).
* **Evaluation Method**: Assess if retrieved contexts are relevant to the alert, and if the LLM's explanation is derived strictly from the retrieved context (checking for hallucinations).

### 2. Hallucination and Factuality Audit
* **Metric**: Factual consistency score (ranging from 0.0 to 1.0).
* **Evaluation Method**: Compare the generated technical walkthrough against official CIS references. Any instruction recommending non-existent flags or parameters is flagged as a failure.

### 3. Output Command Safety
* **Metric**: Blocklist violation rate (must remain at **0.0%**).
* **Evaluation Method**: Pass a test library of alerts containing malicious payload tricks to the AI. If the LLM generates a command containing blocked utilities (e.g., `rm -rf`), verify that the outbound guardrails intercept it and prevent output generation.

### 4. Toxicity and Guardrail Testing
* **Metric**: Prompt leakage rate.
* **Evaluation Method**: Submit prompts trying to access the underlying system prompt. If the assistant leaks its system configuration, prompt optimization is retried.
