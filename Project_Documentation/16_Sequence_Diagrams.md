# 16. Sequence Diagrams

## Introduction

Sequence diagrams illustrate the chronological flow of messages, processing steps, and API calls across different system layers. Below are detailed workflows for alert triaging, conversational queries, and PDF report creation.

---

## 1. Log Ingestion & Threat Analysis Sequence

This sequence traces the path of a security log from detection to user dashboard notification:

```mermaid
sequenceDiagram
    autonumber
    participant Cloud as Cloud Security Provider
    participant GW as Ingestion Gateway
    participant Q as Redis Queue
    participant W as Parsing Worker
    participant DB as Relational Database
    participant VDB as Vector Database
    participant Orch as AI Orchestrator
    participant LLM as LLM Provider API
    participant Guard as Guardrails Shield
    participant UI as Admin Dashboard

    Cloud->>GW: POST /api/v1/ingest (Raw Alert JSON)
    GW->>Q: Enqueue Raw Alert
    GW-->>Cloud: 202 HTTP Accepted
    
    Q->>W: Fetch Alert for Processing
    W->>W: Execute PII Redaction Pipeline
    W->>W: Normalize Log Schema
    W->>DB: Save Alert Status ("PENDING_ANALYSIS")
    
    W->>Orch: Trigger Analysis Event
    Orch->>VDB: Query context matching alert fields
    VDB-->>Orch: Return top-k security controls
    
    Orch->>Orch: Assemble Grounded Prompt
    Orch->>LLM: POST /v1/chat/completions (Sanitized Prompt)
    LLM-->>Orch: Stream Explanations & Mitigation Command Text
    
    Orch->>Guard: Validate response format & command safety
    Guard-->>Orch: Verification Check (PASS)
    
    Orch->>DB: Save Analysis Summary & Update Alert Status ("RESOLVED")
    Orch->>UI: Broadcast Enriched Security Alert (WebSockets)
```

---

## 2. Interactive conversational Query Sequence

This sequence details how a security administrator queries the system for follow-up details:

```mermaid
sequenceDiagram
    autonumber
    participant Admin as Security Admin UI
    participant GW as API Gateway (FastAPI)
    participant Chat as Chat Module
    participant DB as Alert History DB
    participant VDB as Vector Database
    participant Orch as AI Orchestrator
    participant LLM as LLM Provider API
    participant Guard as Guardrails Shield

    Admin->>GW: WebSocket: Send Query ("How do I secure this S3 bucket policy?")
    GW->>Chat: Route Query Request
    Chat->>DB: Retrieve Active Chat History (Session Token)
    DB-->>Chat: Return past 5 messages
    
    Chat->>VDB: Query Vector space for S3 guidelines
    VDB-->>Chat: Return S3 Security Standard documentation chunks
    
    Chat->>Orch: Assemble Query + History + Guidelines
    Orch->>Guard: Check input for Prompt Injection (PASS)
    
    Guard->>LLM: Dispatch Sanitize Prompt
    LLM-->>Orch: Stream response tokens
    
    Orch->>Guard: Audit output CLI command blocks
    Guard-->>Orch: Command Safety Validation (PASS)
    
    Orch->>GW: Stream audited tokens
    GW->>Admin: WebSocket: Stream tokens to Chat Window
```

---

## 3. Incident Report Generation Sequence

This sequence details the compilation and download workflow for incident summaries:

```mermaid
sequenceDiagram
    autonumber
    participant Admin as Security Admin UI
    participant GW as API Gateway (FastAPI)
    participant Rep as Reporting Module
    participant DB as PostgreSQL DB
    participant Audit as Audit Logs DB
    participant Store as Local File Storage

    Admin->>GW: GET /api/v1/reports/generate?incident_id=123
    GW->>Rep: Trigger Report Generation
    
    Rep->>DB: Query Incident Details (Event logs, alert timelines)
    DB-->>Rep: Return Incident Dataset
    
    Rep->>DB: Query AI Analysis & Remediation details
    DB-->>Rep: Return AI context records
    
    Rep->>Rep: Render document templates (Markdown to HTML)
    Rep->>Rep: Compile layout (HTML to PDF conversion)
    
    Rep->>Store: Write compiled PDF file (Secure directory)
    Store-->>Rep: Return access path link
    
    Rep->>Audit: Record event ("Report Generated for incident_id 123")
    Rep-->>GW: Return JSON download url
    GW-->>Admin: Render PDF Download Link on UI
```
