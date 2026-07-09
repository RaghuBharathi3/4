# 32. Project Demo Script

## Demo Overview & Scenario Definition

This document outlines a step-by-step script for demonstrating the **Generative AI-Powered Cloud Security Assistant** during an academic evaluation.

### Incident Scenario: AWS EC2 Credential Harvesting via IMDSv1
* **The Vulnerability**: An EC2 instance runs a web application vulnerable to Server-Side Request Forgery (SSRF).
* **The Exploit**: An attacker uses the SSRF vulnerability to query the Instance Metadata Service (IMDSv1) at `http://169.254.169.254/latest/meta-data/iam/security-credentials/` and harvest IAM credentials.
* **The Trigger**: AWS GuardDuty detects the credential harvesting tool signature and generates an alert JSON payload.

---

## Interactive Demo Timeline

```
+---------------------------------------------------------------------------------+
|                                 Demo Scenario                                   |
|                                                                                 |
|  +--------------------+   +--------------------+   +-------------------------+  |
|  | 1. Trigger Webhook |-->| 2. Dashboard Pop   |-->| 3. Exploit Walks        |  |
|  | - Send GuardDuty   |   | - High Severity    |   | - Natural Language text |  |
|  |   JSON Payload     |   | - Unified Schema   |   | - MITRE Technique T1078 |  |
|  +--------------------+   +--------------------+   +-------------------------+  |
|                                                                |            |
|            +---------------------------------------------------+            |
|            |                                                                |
|            v                                                                |
|  +--------------------+   +--------------------+   +-------------------------+  |
|  | 4. Copy Script     |-->| 5. Chat Query      |-->| 6. PDF Compilation      |  |
|  | - IMDSv2 AWS CLI   |   | - Refine target    |   | - Timelines & signatures|  |
|  | - Terraform IaC    |   |   sub-network zone |   | - Post-Mortem download  |  |
|  +--------------------+   +--------------------+   +-------------------------+  |
+---------------------------------------------------------------------------------+
```

---

## Step-by-Step Execution Script

### Step 1: Simulate the Alert Ingestion
* **Action**: Open a terminal window and run a `curl` command to post a mock GuardDuty log alert payload to the ingestion endpoint:
  ```bash
  curl -X POST http://localhost:8000/api/v1/ingest \
    -H "Content-Type: application/json" \
    -H "X-Signature-SHA256: mock_signature_hash" \
    -d '{
      "provider": "AWS",
      "account": "112233445566",
      "region": "us-east-1",
      "alert_type": "GuardDuty",
      "payload": {
        "id": "gd-999-ec2",
        "type": "UnauthorizedAccess:EC2/MetadataToolPresence",
        "severity": 8.0,
        "resource": {
          "resourceType": "Instance",
          "instanceDetails": {
            "instanceId": "i-09f1a2b3c4d5e"
          }
        }
      }
    }'
  ```
* **Narrative**: *"First, we simulate an alert ingress event. A mock AWS GuardDuty payload is posted to our ingestion endpoint, simulating the detection of credential harvesting tools on a virtual machine."*

### Step 2: Dashboard Alert Card Popup
* **Action**: Show the Next.js dashboard UI. The alerts table dynamically refreshes, displaying a **High Severity** alert card at the top of the list.
* **Narrative**: *"On the dashboard, the normalized alert card appears in real-time. Notice how the raw JSON has been parsed and displayed as a clean card showing the severity level, cloud provider, and affected resource ID."*

### Step 3: Exploit Explanation & MITRE Mapping
* **Action**: Click the alert card. The sidebar expands to display the AI-generated triage report.
* **Narrative**: *"Clicking the alert card displays the AI-generated report. The assistant explains in plain language that an attacker is trying to harvest credentials from an EC2 instance by exploiting a vulnerability to query the metadata service. It also maps the activity to MITRE ATT&CK technique T1078."*

### Step 4: Reviewing Remediation Scripts
* **Action**: Scroll down to display the code blocks containing the AWS CLI script and Terraform patch suggestion.
* **Narrative**: *"The assistant provides a copyable CLI command to mitigate the threat: `aws ec2 modify-instance-metadata-options --instance-id i-09f1a2b3c4d5e --http-tokens required --http-endpoint enabled`. This command enforces IMDSv2, requiring a session token to query metadata and blocking the credential harvesting attempt. It also provides a Terraform snippet to apply the fix as code."*

### Step 5: Interactive Chat Refinement
* **Action**: Open the chat window and type: *"Can you change the CLI command to include the specific AWS profile named 'prod-security'?"*
* **Narrative**: *"If we want to adjust the remediation advice, we can use the chat interface. I will ask the assistant to include a specific AWS profile. The assistant streams back an updated command: `aws ec2 modify-instance-metadata-options --instance-id i-09f1a2b3c4d5e --http-tokens required --http-endpoint enabled --profile prod-security`."*

### Step 6: Post-Incident Report Download
* **Action**: Click the "Generate Incident Report" button. Wait for a few seconds, then click the download link to open a clean PDF file containing the incident summary, timeline, and mitigation steps.
* **Narrative**: *"Finally, with one click, we compile a post-incident report. The PDF documents the incident timeline, the AI's analysis, the recommended remediation steps, and the audit trail logs, providing an archive for compliance reviews."*
