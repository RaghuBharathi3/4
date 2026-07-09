# 19. Use Case Diagrams

## Introduction

The Use Case diagram identifies the boundaries of the **Generative AI-Powered Cloud Security Assistant**, mapping interactions between system actors (Security Administrator, Auditor, and Webhook Agent) and the core features of the application.

---

## System Use Case Diagram

The diagram below details the operational boundaries and associated use cases:

```mermaid
graph TD
    %% Actors
    Admin((Security Administrator))
    Auditor((Security Auditor))
    Webhook((Webhook Ingress Agent))

    subgraph "Cloud Security Assistant System Boundary"
        UC1(Ingest & Sanitize JSON logs)
        UC2(View Real-Time Threat Dashboard)
        UC3(Ask Questions in Chat Interface)
        UC4(Retrieve Remediation Scripts)
        UC5(Verify & Copy CLI commands)
        UC6(Generate post-mortem PDF Reports)
        UC7(Perform Semantic Log Search)
        UC8(Modify AI Prompt Configurations)
        UC9(Inspect WORM Audit Records)
    end

    %% Webhook Connections
    Webhook --> UC1

    %% Administrator Connections
    Admin --> UC2
    Admin --> UC3
    Admin --> UC4
    Admin --> UC5
    Admin --> UC6
    Admin --> UC7
    Admin --> UC8

    %% Auditor Connections
    Auditor --> UC2
    Auditor --> UC6
    Auditor --> UC7
    Auditor --> UC9

    %% Use Case dependencies (includes / extends)
    UC3 -.->|includes| UC7
    UC4 -.->|includes| UC5
    UC6 -.->|includes| UC9
```

---

## Actor Definitions

### 1. Security Administrator
* **Role**: Primary operator responsible for resolving threats, managing configuration, and maintaining cloud posture.
* **Interactions**:
  * Monitors dashboards to detect active incidents.
  * Conversations with the AI to refine remediation actions.
  * Reviews and copy-pastes CLI scripts to mitigate security risks.
  * Customizes prompting logic to align analysis outputs with company policies.

### 2. Security Auditor
* **Role**: Compliance assessor responsible for verifying posture standards and reviewing security records.
* **Interactions**:
  * Evaluates historical logs using semantic search queries.
  * Generates PDF compliance summaries for external audits.
  * Audits system access history and AI outcomes to verify integrity.

### 3. Webhook Ingress Agent (System Actor)
* **Role**: Automated worker representing external event brokers (e.g., AWS EventBridge, Azure Event Grid).
* **Interactions**:
  * Pushes security events and alert payloads to Ingestion endpoints.
  * Triggers threat interpretation queues.
