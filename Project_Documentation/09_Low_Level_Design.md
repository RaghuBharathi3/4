# 09. Low-Level Design

## Introduction

The Low-Level Design (LLD) provides developer-centric details for the components of the **Generative AI-Powered Cloud Security Assistant**. It covers schema interfaces, prompt template configurations, input redaction algorithms, and RAG search indexing mechanics.

---

## 1. Normalized Log Telemetry Schema

To support multi-cloud log inputs, all source JSON alerts are normalized into a unified class structure. Below is the structural representation (defined conceptually in Python Pydantic format):

```python
from pydantic import BaseModel, Field
from typing import Dict, Any, Optional
from datetime import datetime

class NormalizedEvent(BaseModel):
    event_id: str = Field(..., description="Unique hash generated from the raw event contents")
    timestamp: datetime = Field(..., description="UTC timestamp of the log event creation")
    cloud_provider: str = Field(..., description="AWS, AZURE, or GCP")
    account_id: str = Field(..., description="Source cloud account or subscription ID")
    region: str = Field(..., description="Deployment region where the event occurred")
    
    # Actor/Attacker Information
    actor_identity: str = Field(..., description="Assumed IAM user, Service Account, or API principal")
    actor_type: str = Field(..., description="User, AssumedRole, ServiceAccount, or Unknown")
    source_ip: str = Field(..., description="Source IP address or redacted placeholder")
    
    # Resource Information
    target_resource_arn: str = Field(..., description="Full ARN, Resource ID, or self-link of target asset")
    resource_type: str = Field(..., description="EC2, S3_Bucket, VirtualMachine, IAM_Role, etc.")
    
    # Exploit Action
    event_name: str = Field(..., description="API call or trigger action (e.g. AuthorizeSecurityGroupIngress)")
    severity: str = Field(..., description="CRITICAL, HIGH, MEDIUM, LOW")
    
    # Metadata Payload
    raw_payload_cache: Dict[str, Any] = Field(..., description="PII-redacted original JSON snippet")
```

---

## 2. Prompt Engineering & Templates

The assistant relies on carefully structured prompts to enforce consistency and prevent hallucinations. The prompt follows a role-definition, grounding-context, alert-metadata, and output-format layout:

```
================================================================================
SYSTEM PROMPT (ROLE DEFINITION)
================================================================================
You are an expert Cloud Security Incident Response Architect. Your job is to analyze 
normalized cloud security alerts, cross-reference them with the provided Grounding 
Context, and construct a precise, clear threat explanation and remediation plan.

Under no circumstances should you generate remediation commands that contain wildcards (*) 
in critical write-operations, nor should you assume the presence of unspecified scripts.
If the provided context does not contain enough information to resolve the incident, 
explicitly state the missing information.

================================================================================
GROUNDING CONTEXT (FROM VECTOR DB / RAG)
================================================================================
[CIS Benchmark AWS Section 5.1]: "Ensure no security groups allow unrestricted ingress to port 22."
[AWS Documentation CLI guide]: "To revoke security group ingress, use: 
aws ec2 revoke-security-group-ingress --group-id <group_id> --protocol tcp --port 22 --cidr <cidr>"

================================================================================
NORMALIZED EVENT telemetries (JSON)
================================================================================
{
  "cloud_provider": "AWS",
  "account_id": "112233445566",
  "region": "us-east-1",
  "actor_identity": "arn:aws:iam::112233445566:user/admin-temp",
  "source_ip": "198.51.100.12",
  "target_resource_arn": "sg-01a2b3c4d5e6f7g8h",
  "event_name": "AuthorizeSecurityGroupIngress",
  "severity": "HIGH"
}

================================================================================
RESPONSE FORMAT SPECIFICATION
================================================================================
Respond ONLY in valid Markdown using the following structure:

### 1. Incident Executive Summary
[A concise summary explaining what happened in plain language]

### 2. Exploited Vectors & MITRE ATT&CK Mapping
* **Tactic**: [e.g., Initial Access / Credential Access]
* **Technique**: [e.g., T1190 - Exploit Public-Facing Application]

### 3. Step-by-Step Technical Explanation
[Detailed walkthrough of how the action affects resources]

### 4. Direct Remediation Command
```bash
[Exact CLI execution command containing variables matching target resource IDs]
```

### 5. long-term prevention (Terraform/IaC example)
```hcl
[Terraform code to enforce secure posture]
```
================================================================================
```

---

## 3. RAG Retrieval & Vector Indexing Specifications

To ground the model's outputs, documentation is processed through the following pipeline:

### Document Chunking Strategy
* **Method**: Recursive Character Text Splitter.
* **Chunk Size**: 512 tokens.
* **Chunk Overlap**: 52 tokens (approximately 10% overlap to preserve context across boundaries).

### Vector Database Configuration
* **Embedding Model**: `text-embedding-3-small` (1536 dimensions) or Google `text-embedding-004` (768 dimensions).
* **Distance Metric**: **Cosine Similarity** to match semantic meaning independently of document length:
  $$\text{Similarity}(\mathbf{A}, \mathbf{B}) = \frac{\mathbf{A} \cdot \mathbf{B}}{\|\mathbf{A}\| \|\mathbf{B}\|}$$

---

## 4. Input Redaction & Sanitization Regex

The **PII Redactor** evaluates strings using a sequential regex pipeline. Below is the list of filters run prior to any LLM API invocation:

```python
# Regular expression rules for deterministic PII scraping
REDACTION_PIPELINE = [
    # AWS Access Key IDs
    (r"(?<![A-Z0-9])AKIA[A-Z0-9]{16}(?![A-Z0-9])", "<REDACTED_AWS_ACCESS_KEY>"),
    # AWS Secret Access Keys
    (r"(?<![A-Za-z0-9+/])[A-Za-z0-9+/]{40}(?![A-Za-z0-9+/])", "<REDACTED_AWS_SECRET_KEY>"),
    # Generic Bearer Tokens
    (r"bearer\s+[A-Za-z0-9\-\._~\+\/]+=*", "bearer <REDACTED_TOKEN>"),
    # IPv4 Addresses (Excluding local loopbacks and documentation IPs)
    (r"\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b", lambda match: redact_ip(match.group())),
    # Private Email Addresses
    (r"[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+", "<REDACTED_EMAIL_ADDRESS>")
]
```

---

## 5. Output Verification Engine (Syntax Shield)

Before the generated response is broadcast, the script block is parsed by a verification gate:

```python
def verify_output_safety(raw_llm_response: str) -> bool:
    # 1. Check for basic script injection inside code blocks
    if "sudo rm -rf /" in raw_llm_response:
        return False
        
    # 2. Prevent wildcards in IAM actions that grant open permissions
    if '"Action": "*"' in raw_llm_response and '"Effect": "Allow"' in raw_llm_response:
        return False
        
    # 3. Check for obvious syntax placeholders that would break terminals
    if "<your_" in raw_llm_response or "[insert_" in raw_llm_response:
        return False
        
    return True
```
