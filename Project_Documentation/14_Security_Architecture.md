# 14. Security Architecture

## Overview

The security architecture of the **Generative AI-Powered Cloud Security Assistant** is built on the principle of **Defense in Depth**. Because this system handles sensitive cloud infrastructure logs and generates operational commands, it must be hardened against data leakage, malicious prompt injections, and unauthorized actions.

---

## Security Architecture Blueprint

```
       [Raw JSON Alert Data]          [User Conversational Input]
                 |                                |
                 v                                v
    +--------------------------+    +--------------------------+
    | Inbound Regex Redactor   |    | Prompt Injection Shield  |
    | (Strips access keys, PII)|    | (Blocks overrides)       |
    +------------+-------------+    +------------+-------------+
                 |                               |
                 +---------------+---------------+
                                 | (Sanitized Inputs)
                                 v
                    +--------------------------+
                    |    AI Orchestrator API   |
                    +------------+-------------+
                                 | (Prompt execution)
                                 v
                    +--------------------------+
                    |    LLM Inference Engine  |
                    +------------+-------------+
                                 | (Raw AI Output)
                                 v
    +----------------------------------------------------------+
    |                Outbound Guardrails Engine                |
    |   +--------------------------+   +-------------------+   |
    |   | Output Command Blocklist |   | Syntax Linter &   |   |
    |   | (Prevents rm -rf, etc.)  |   | Wildcard Check    |   |
    |   +--------------------------+   +-------------------+   |
    +----------------------------+-----------------------------+
                                 | (Verified Outputs)
                                 v
                    +--------------------------+
                    |   Audit Logs Database    | (Write-Once-Read-Many)
                    +------------+-------------+
                                 | (Rendered UI Component)
                                 v
                    [Administrator Console UI]
```

---

## 1. Least Privilege Access Control

The assistant requires access to cloud logs, metadata databases, and the vector index. However, it operates on a read-only architecture:
* **Infrastructure Read-Only Policies**: The service credentials (IAM Roles or Service Principals) assigned to the assistant are strictly configured with read-only permissions (e.g., `SecurityAudit` or `ReadOnlyAccess`). Under no circumstances are write policies (`iam:*`, `ec2:RunInstances`, or `s3:PutBucketPolicy`) attached to the assistant's credentials.
* **Database Isolation**: The backend connection pools partition write privileges. Only the Log Ingest component is authorized to write to the alert database; the AI reasoning module is restricted to read-only access for alert records.

---

## 2. Inbound Sanitization & PII Redaction

To prevent sensitive enterprise configuration details from leaking to external public APIs (such as OpenAI or Anthropic endpoints), the system runs an **Inbound Sanitization pipeline**:
* **Credential Stripping**: High-entropy strings matching cloud API credential structures (AWS Access Key ID, AWS Secret Access Key, GCP Service Account Private Keys) are replaced with dummy tokens.
* **Deterministic IP Generalization**: External IPv4 and IPv6 addresses are mapped to virtual markers (e.g., `198.51.100.23` becomes `IPv4_HOST_1`), hiding the specific target server infrastructure.
* **User Identity Obfuscation**: Personal names, domain-specific emails, and local network directories are replaced with standard system placeholders.

---

## 3. Adversarial Prompt Injection Mitigation

Prompt injection occurs when an attacker inputs malicious text to override the LLM's system instructions. The assistant implements three mitigation strategies:
* **System Prompt Isolation**: The system prompt is kept separate from user inputs in API requests using structured payloads (e.g., system instructions objects within Anthropic and Google SDK calls), which models prioritize over user message structures.
* **Heuristic Injection Scanning**: Interactive chat queries are pre-scanned for common injection strings (e.g., "ignore all previous instructions", "reprogram yourself", "new rule:").
* **LLM-Based Intent Classification**: High-risk prompts are routed to a local small model to classify their intent before they reach the main orchestration engine. If the prompt is flagged as an override attempt, it is rejected.

---

## 4. Outbound Guardrails & Command Verification

Before rendering recommended shell scripts (AWS CLI, Azure CLI, GCP gcloud) or Infrastructure as Code modifications to the administrator, the outputs must pass an **Outbound Verification check**:
* **Structural Code Linter**: Validates that all generated code blocks compile cleanly, preventing syntax errors that could cause script executions to fail mid-operation.
* **Command Blocklist**: Denies any commands containing dangerous scripts:
  * Operations altering system files (`/etc/`, `/var/`).
  * System deletion operations (`rm -rf`, `format`, `dd`).
  * Unauthorized network tools (`netcat`, `nmap`, `iptables` overrides).
* **Principal Wildcard Auditing**: Denies policy modifications containing open wildcard actions (e.g., `"Action": "*"` with `"Principal": "*"`).

---

## 5. Write-Once-Read-Many (WORM) System Auditing

Every interaction with the assistant is securely documented to prevent trace erasure:
* **Action Logs**: The system records the timestamp, user ID, raw prompt, retrieved context files, generated AI response, and the exact script blocks copied to the clipboard.
* **Storage Standard**: These logs are stored in a dedicated database collection configured with Write-Once-Read-Many (WORM) constraints, preventing attackers who compromise the application server from altering the audit history.

---

## 6. Network Isolation & Security Group Boundaries

* **Virtual Private Cloud (VPC)**: The backend API services, vector database instances, and caching servers are deployed within private subnets in the organization's cloud VPC.
* **Private API Endpoints**: Outgoing calls to LLM provider APIs use private network links (such as AWS PrivateLink or Azure Private Link) where available, ensuring traffic does not touch the public internet.
* **Web Application Firewall (WAF)**: The frontend dashboard is protected behind a Cloud WAF, blocking DDoS attempts and unauthorized IP networks from hitting the console.
