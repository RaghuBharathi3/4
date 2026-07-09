# 23. API Design (Conceptual Only)

## Introduction

This document outlines the conceptual API architecture for the **Generative AI-Powered Cloud Security Assistant**. It details the RESTful and WebSockets endpoints used for log ingestion, alert triaging, conversational queries, and report generation, along with their request/response schemas.

---

## 1. RESTful Ingestion & Management APIs

### Ingest Security Webhook Alert
* **Endpoint**: `POST /api/v1/ingest`
* **Purpose**: Target webhook url for incoming cloud alerts.
* **Authentication**: Hmac signature in headers (`X-Signature-SHA256`).
* **Request Payload (JSON)**:
```json
{
  "provider": "AWS",
  "account": "123456789012",
  "region": "us-east-1",
  "alert_type": "GuardDuty",
  "payload": {
    "id": "gd-1a2b-3c4d",
    "type": "UnauthorizedAccess:EC2/MetadataToolPresence",
    "severity": 8.0,
    "resource": {
      "resourceType": "Instance",
      "instanceDetails": {
        "instanceId": "i-09f1a2b3c4d5e"
      }
    },
    "service": {
      "action": {
        "actionType": "PORT_PROBE",
        "portProbeAction": {
          "portProbeDetails": [{"localPortSpec": {"port": 80}}]
        }
      }
    }
  }
}
```
* **Response Payload (JSON - HTTP 202 Accepted)**:
```json
{
  "status": "ACCEPTED",
  "event_id": "9f8e7d6c-5b4a-3f2e-1d0c-9b8a7f6e5d4c",
  "received_at": "2026-07-03T15:08:12Z"
}
```

### Retrieve Incident Triage Details
* **Endpoint**: `GET /api/v1/alerts/{incident_id}`
* **Purpose**: Fetches metadata and compiled AI reports for a specific incident.
* **Authentication**: Bearer JWT Token.
* **Response Payload (JSON - HTTP 200 OK)**:
```json
{
  "incident_id": "9f8e7d6c-5b4a-3f2e-1d0c-9b8a7f6e5d4c",
  "status": "RESOLVED",
  "severity": "HIGH",
  "timestamp": "2026-07-03T15:08:12Z",
  "log_summary": {
    "cloud_provider": "AWS",
    "event_name": "MetadataToolPresence",
    "target_resource": "i-09f1a2b3c4d5e"
  },
  "ai_report": {
    "summary": "An EC2 instance is querying its own metadata service using tools associated with credential harvesting.",
    "exploit_explanation": "An attacker compromising the application running on i-09f1a2b3c4d5e is attempting to pull IAM credentials from the Instance Metadata Service (IMDSv1) to escalate privileges.",
    "remediation": {
      "cli_command": "aws ec2 modify-instance-metadata-options --instance-id i-09f1a2b3c4d5e --http-tokens required --http-endpoint enabled",
      "remediation_type": "IMDSv2_Enforcement"
    }
  }
}
```

---

## 2. WebSockets Conversational APIs

Real-time, bidirectional chat streams route through full-duplex WebSockets to stream LLM responses to administrators.

### Chat Session Connection
* **Endpoint**: `WS /api/v1/chat/{session_id}`
* **Parameters**:
  * `session_id`: UUID matching the active chat session.
* **Headers**:
  * `Authorization`: `Bearer <JWT_TOKEN>`

### Client Message Inbound Schema
```json
{
  "action": "SEND_MESSAGE",
  "content": "Can you format the remediation command as a Terraform configuration block instead?"
}
```

### Server Response Token Streaming Schema
```json
{
  "event": "TOKEN_STREAM",
  "token": "resource ",
  "index": 12
}
```

### Server Response Message Completed Schema
```json
{
  "event": "MESSAGE_COMPLETE",
  "full_content": "Here is the Terraform configuration block to enforce IMDSv2 on your instances:\n\n```hcl\nresource \"aws_instance\" \"example\" {\n  # ...\n  metadata_options {\n    http_endpoint = \"enabled\"\n    http_tokens   = \"required\"\n  }\n}\n```",
  "citations": [
    {
      "source_title": "CIS AWS Foundation Benchmark v1.4 Securing EC2",
      "url": "https://docs.aws.amazon.com/securityhub/latest/userguide/ec2-controls.html"
    }
  ]
}
```

---

## 3. API Security & Rate Limits

To protect system integrity and manage LLM API costs:
* **JWT Access tokens**: Issued upon user login, validating the user's role (`security_admin` or `read_only_analyst`).
* **Cross-Origin Resource Sharing (CORS)**: Strict white-listing allowing API calls only from designated enterprise console origins.
* **Rate Limits**:
  * `/api/v1/ingest`: Scaled limit of **1,000 requests per minute** (throttled using Redis token bucket algorithms).
  * WebSockets chat queries: Limit of **10 conversational completions per minute** per administrator to prevent API abuse.
