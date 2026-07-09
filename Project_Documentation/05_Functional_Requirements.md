# 05. Functional Requirements

## Introduction

Functional requirements define the core operations, user interactions, and information processing tasks that the **Generative AI-Powered Cloud Security Assistant** must support. These are organized using the **MoSCoW** (Must have, Should have, Could have, Won't have) prioritization system.

---

## MoSCoW Prioritization Matrix

```
                      +---------------------------------------+
                      |         Functional Scope              |
                      +---------------------------------------+
                                          |
        +---------------------------------+---------------------------------+
        |                                                                   |
        v                                                                   v
+-------------------------------+                                   +-------------------------------+
|          MUST HAVE            |                                   |          SHOULD HAVE          |
|  - Log Ingestion Adapters     |                                   |  - Incident Timelines         |
|  - Threat Translation Engine  |                                   |  - PDF / MD Reporting         |
|  - RAG Documentation Lookup   |                                   |  - RBAC (Admin, Analyst)      |
|  - Remediation Scripts        |                                   |  - Alert Threshold Policies   |
|  - Interactive Chat Interface |                                   +-------------------------------+
|  - PII Redactor / Guardrails  |                                                   |
+-------------------------------+                                                   |
        |                                                                           |
        +---------------------------------+-----------------------------------------+
                                          |
        +---------------------------------+---------------------------------+
        |                                                                   |
        v                                                                   v
+-------------------------------+                                   +-------------------------------+
|          COULD HAVE           |                                   |          WON'T HAVE           |
|  - Multi-Agent Orchestrator   |                                   |  - Auto-Execution of Scripts  |
|  - Chatbot Webhook Alerts     |                                   |  - Direct Console API Writes  |
|  - Static Syntax Linter       |                                   |  - Host-level agent setups    |
+-------------------------------+                                   +-------------------------------+
```

---

## Detailed Requirement Specifications

### 1. Ingestion & Pre-processing (Log Ingestion Module)
* **FR-1.1**: The system **MUST** accept security JSON payloads from AWS GuardDuty, AWS CloudTrail, Microsoft Defender for Cloud, and GCP Security Command Center.
* **FR-1.2**: The system **MUST** normalize inputs into a common telemetry schema:
  `{ incident_id, timestamp, cloud_provider, source_identity, source_ip, target_resource_arn, event_name, raw_severity, event_category }`.
* **FR-1.3**: The system **MUST** redact sensitive information (AWS Access Key IDs, Secret Keys, database connection strings, passwords, OAuth tokens) using Regex rules and Named Entity Recognition (NER) prior to LLM processing.

### 2. AI Reasoning & Threat Analysis (AI Reasoning Module)
* **FR-2.1**: The system **MUST** interpret raw event parameters and output:
  1. A natural language executive summary (suitable for non-technical managers).
  2. A technical exploit walkthrough (explaining the threat actor's methodology).
  3. Risk level mapping (Low, Medium, High, Critical) based on incident characteristics.
* **FR-2.2**: The system **MUST** map identified event tags to specific **MITRE ATT&CK** Tactics and Techniques (e.g., *TA0006 - Credential Access*, *T1078 - Valid Accounts*).

### 3. Retrieval-Augmented Generation (RAG Core)
* **FR-3.1**: The system **MUST** index compliance guidelines (CIS Controls, SOC2 Security Criteria) and cloud documentation.
* **FR-3.2**: When responding to chat queries or explaining alerts, the system **MUST** query the Vector Database to enrich context.
* **FR-3.3**: The system **MUST** append verifiable footnotes or hyperlink references to the official cloud documentation source used for generating the advice.

### 4. Remediation Guidance & Action Plans (Remediation Module)
* **FR-4.1**: The system **MUST** formulate targeted remediation scripts in:
  * **Shell Scripting**: CLI commands (AWS CLI, Azure CLI, gcloud CLI) to isolate resources, revoke sessions, or update security rules.
  * **Infrastructure as Code (IaC)**: Terraform, CloudFormation, or Ansible code changes to fix structural misconfigurations.
* **FR-4.2**: The system **MUST** ensure all generated code scripts are isolated inside read-only interactive display components in the dashboard interface to prevent automatic browser execution.

### 5. Conversation & Search Workspace (Chat Module)
* **FR-5.1**: The system **MUST** support multi-turn conversational chat, allowing admins to refine generated remediation plans (e.g., "Change the command to apply to subnet-2 instead of subnet-1").
* **FR-5.2**: The system **MUST** enable semantic search over past alert summaries and incident records using natural language queries (e.g., "Find all IAM escalation events from last Tuesday").

### 6. Post-Incident Reporting (Reporting Module)
* **FR-6.1**: The system **SHOULD** construct automated Incident Post-Mortem reports outlining:
  * Event Detection Timeline.
  * Exploited Vulnerability details.
  * Impacted Infrastructure components.
  * Completed Remediation actions.
  * Long-term Recommendations.
* **FR-6.2**: The system **SHOULD** enable one-click export of summaries into PDF or Markdown files.

### 7. Administration & System Management
* **FR-7.1**: The system **SHOULD** enforce Role-Based Access Control (RBAC):
  * **Security Administrator**: Full rights to view logs, request AI plans, export reports, and adjust prompt configurations.
  * **Read-Only Analyst**: Authorized to search alerts and chat with the assistant but cannot change prompt configurations or ingestion rules.
* **FR-7.2**: The system **SHOULD** log all system actions (user logins, LLM queries, configurations updated) to a tamper-proof database log file.
* **FR-7.3**: The system **COULD** support outgoing webhooks to Slack or MS Teams channel dashboards when high-severity threats are successfully translated.
