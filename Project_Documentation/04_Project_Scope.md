# 04. Project Scope

## In-Scope Capabilities (Included Features)

The **Generative AI-Powered Cloud Security Assistant** encompasses the design and architectural boundaries outlined below. The implementation plan focuses on the following primary functional blocks:

### 1. Multi-Cloud Log Adapter & Parser
* Ingestion of raw alert payloads from:
  * **AWS**: GuardDuty alerts and CloudTrail event logs.
  * **Azure**: Microsoft Defender for Cloud alerts and Activity logs.
  * **GCP**: Security Command Center findings and Cloud Audit logs.
* Normalization engine to parse varied input formats (JSON) into a unified internal security schema (specifying Actor, Source IP, Target Resource, Operation, Timestamp, Provider, Severity).

### 2. AI Reasoning & Threat Translation Engine
* Integration with enterprise-grade LLMs (via API or local hosting) to translate parsed events into human-readable narratives.
* Generation of detailed step-by-step remediation plans tailored to the specific context of the alert (e.g., target resource ID, region, account number).
* Dynamic prompt engineering modules featuring few-shot examples of cloud security triage patterns.

### 3. Retrieval-Augmented Generation (RAG) System
* Embedding and indexing of essential cybersecurity knowledge bases:
  * Official vendor documentation (AWS Securing Resources, Azure Security Guidelines, GCP Architecture Framework).
  * Industry standard compliance controls (CIS Benchmarks, MITRE ATT&CK Matrix).
  * Threat intelligence files (Common Vulnerabilities and Exposures - CVE list).
* Semantic search utilities enabling administrators to query the system for security advice and receive contextually verified answers.

### 4. Conversational Security Dashboard
* A browser-based administrative interface providing:
  * A real-time incident monitoring dashboard grouping logs by severity and category.
  * An interactive chat assistant to ask questions regarding specific alerts, historical trends, or cloud architectures.
  * Interactive CLI preview components to copy, verify, and document proposed remediation scripts.

### 5. Report Generation Engine
* Export utilities to construct detailed, post-incident analysis documents (in Markdown or PDF format).
* Custom dashboard modules providing aggregated compliance views (e.g., "Weekly posture against CIS benchmarks").

### 6. Security Guardrails & Safety Audits
* Input parsing algorithms to redact PII (Passwords, access keys, email addresses) before sending log contents to external LLM APIs.
* Adversarial prompt injection screening (Prompt Shielding).
* Execution constraints ensuring all mitigation commands are displayed to the user for explicit approval rather than executed silently.

---

## Out-of-Scope (Excluded Capabilities)

To maintain project focus, feasibility, and system safety, the following items are explicitly excluded from this design phase:

### 1. Autonomous Remediation Execution (Active Defense)
* **Description**: The system **will not** automatically execute CLI scripts or modify live cloud infrastructure (e.g., it will not run an AWS CLI command to delete an IAM policy or shut down an EC2 instance directly).
* **Rationale**: Giving an AI model write access to modify live infrastructure without human confirmation is a significant operational hazard. The model could misinterpret a benign log and disable a production service.

### 2. High-Throughput Log Aggregation and Storage (SIEM Replacement)
* **Description**: The system is not designed to act as a scalable data lake (like Splunk, Snowflake, or AWS S3 data lakes) to archive terabytes of hot/cold infrastructure telemetry.
* **Rationale**: The assistant assumes that logs are already gathered, filtered, and alerted on by primary cloud engines. The assistant ingests only targeted alerts and filtered streams, keeping storage demands lightweight.

### 3. Host-Level Intrusion Detection System (EDR/XDR)
* **Description**: The system will not deploy OS-level tracking agents (e.g., Falco, OSSEC, Wazuh agents) inside target VMs, virtual machines, or Kubernetes nodes.
* **Rationale**: Host-level telemetry collection is managed by standard security tools. The assistant relies on cloud API audit streams and pre-existing alert triggers.

### 4. Custom Foundation Model Fine-Tuning
* **Description**: The design does not involve pre-training or fine-tuning foundation models (e.g., training a raw Llama model from scratch on security datasets).
* **Rationale**: Modern prompt engineering, multi-agent frameworks, and RAG architectures provide superior accuracy, easier updates, and lower cost compared to training custom weights.

---

## Scope Matrix

| Component | In-Scope Details | Out-of-Scope Details |
| :--- | :--- | :--- |
| **Ingestion** | Parses JSON alerts from GuardDuty, Azure Defender, and GCP SCC. | Ingesting raw network VPC flows or high-throughput system calls. |
| **AI Processing** | Prompt construction, threat analysis, explanation generation. | Training custom foundation security LLMs from raw weights. |
| **Action Plan** | Recommending specific remediation CLI scripts and Terraform code. | Automated execution of commands on live production infrastructure. |
| **Storage** | Database storing parsed alerts, vector representation of guidelines. | Multi-terabyte log aggregation, cold archiving, compliance data vaults. |
| **Interface** | Admin Chat UI, reporting layouts, dashboard statistics. | Complete cloud configuration editor, multi-account management console. |
