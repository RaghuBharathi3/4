# 08. High-Level Design

## Introduction

The High-Level Design (HLD) outlines the subsystem interfaces, functional workflows, and data boundaries of the **Generative AI-Powered Cloud Security Assistant**. It details how raw alert telemetry is ingested, sanitized, enriched with external knowledge, processed by the LLM, and finally made available to security administrators via clean web workspaces.

---

## Subsystem Interface Overview

```
                      +---------------------------------------+
                      |         Log Ingestion Gateway         |
                      +---------------------------------------+
                                          | (Sanitized JSON)
                                          v
                      +---------------------------------------+
                      |       Central AI Orchestrator         |
                      +---------------------------------------+
                               /                     \
       (Retrieve Knowledge)  /                         \  (LLM Prompt/Response)
                            v                           v
              +-----------------------+       +-------------------+
              |  Vector DB / RAG Core |       |  Remote LLM API   |
              +-----------------------+       +-------------------+
                            \                           /
                 (Context)   \                         /  (AI Output)
                              v                       v
                      +---------------------------------------+
                      |        Safety Guardrails Engine       |
                      +---------------------------------------+
                                          | (Verified Explanation & Script)
                                          v
                      +---------------------------------------+
                      |        FastAPI / presentation         |
                      +---------------------------------------+
```

---

## Processing Pipeline Phases

### Phase 1: Alert Intake and Normalization
1. **Webhook Callback**: Cloud provider alerts (e.g., AWS GuardDuty AWS Security Hub alert) hit the public HTTPS `/api/v1/ingest` endpoint of the Log Ingestion Gateway.
2. **Buffer Queue**: The API verifies the basic signature and immediately pushes the payload to the Redis queue (`alert_queue`) returning HTTP 202 Accepted. This guarantees minimal latency and prevents connection exhaustion.
3. **Parsing Worker**: A background worker process pops the alert from the queue, executes regex-based PII redaction, maps the attributes to a standardized log schema, and saves the record in the relational database metadata store as "PENDING_ANALYSIS".

### Phase 2: Context Enrichment (RAG)
1. **Query Construction**: The orchestrator extracts key fields from the normalized log (such as the event name e.g., `UpdateAssumeRolePolicy` and the warning category e.g., `PrivilegeEscalation:IAMUser`).
2. **Semantic Retrieve**: The query is vectorized and run against the Vector Database. The system retrieves:
   * **CIS Reference Guidelines**: Specific steps detailing secure configurations for that resource.
   * **Threat Intel Records**: Specific known CVE files matching the services mentioned.
3. **Context Compilation**: The retrieved text blocks are compiled into a list of structured references, which will serve as grounding context for the LLM to prevent hallucinations.

### Phase 3: Prompt Assembly and LLM Inference
1. **Template Selection**: The orchestrator retrieves the prompt template designed for the matching category (e.g., IAM Exploit template, Network Exploit template).
2. **Prompt Hydration**: The prompt template is populated with the normalized event logs, the RAG context, and historical context of the active conversation.
3. **Guardrails Audit (Inbound)**: The hydrated prompt is checked for potential user injection overrides.
4. **LLM Query**: The sanitized prompt is dispatched to the chosen LLM API endpoint. The model streams tokens back in real-time.
5. **Guardrails Audit (Outbound)**: The generated explanation and recommended mitigation scripts are analyzed for safe scripting patterns.

### Phase 4: Presentation and Interaction
1. **WebSocket Broadcast**: The verified, structured AI output is sent over a WebSocket connection to the Next.js frontend application, triggering an active alert notification.
2. **Interactive Workspace**: The administrator views the plain-language walkthrough, visualizes the threat timeline, copy-pastes the verified remediation scripts, and asks follow-up questions to refine the mitigation commands.
3. **Feedback Tracking**: If the administrator copies a CLI command, the system logs the event in the audit trail, tracking mitigation velocity.

---

## Internal Subsystem Communications

* **Gateway-to-Worker**: Async messaging using Redis pub/sub queues.
* **Orchestrator-to-Databases**: Relational database connection pools (using PostgreSQL SQLAlchemy/Prisma) and vector search API calls.
* **Orchestrator-to-LLM**: Secured HTTPS requests targeting official API endpoints (using SDKs like `google-generativeai` or `anthropic`).
* **Frontend-to-Backend**: REST APIs for administrative queries and historical lookups; WebSockets for real-time threat stream delivery.
