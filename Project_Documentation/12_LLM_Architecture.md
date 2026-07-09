# 12. LLM Architecture

## Model Selection Framework

Choosing the correct Large Language Model (LLM) for a cloud security assistant requires balancing security intelligence, context window limits, token costs, processing latency, and data privacy options.

---

## Detailed Model Comparison Table

| Model Options | Provider | Est. Input Cost (per 1M tokens) | Est. Output Cost (per 1M tokens) | Context Window | Offline / Local Capability | Primary Strength | Weakness |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Gemini 1.5 Pro** | Google | \$1.25 (under 128k) | \$5.00 (under 128k) | 2,000,000 | No | Enormous context window; superior JSON parsing and multimodal log mapping. | Slightly higher latency on initial tokens. |
| **Gemini 1.5 Flash** | Google | \$0.075 (under 128k) | \$0.30 (under 128k) | 1,000,000 | No | Extremely fast token execution; very low cost. Ideal for real-time triage. | Lower complex reasoning accuracy compared to Pro. |
| **Claude 3.5 Sonnet** | Anthropic | \$3.00 | \$15.00 | 200,000 | No | Industry-leading coding capabilities, precise script generation, high logic accuracy. | Higher token cost. |
| **GPT-4o** | OpenAI | \$2.50 | \$10.00 | 128,000 | No | Strong general logic; fast responses; extensive enterprise integrations. | Strict data policies; proprietary closed ecosystem. |
| **Llama 3 (70B)** | Meta | Free (open-weight) | Free (open-weight) | 8,000 | **Yes** (via local vLLM / Ollama) | 100% data privacy; zero API dependency; high control over parameters. | Requires dedicated GPU hardware; complex local infrastructure. |
| **Mistral Large** | Mistral AI | \$2.00 | \$6.00 | 128,000 | No | Strong multilingual capabilities; European data hosting and privacy options. | Smaller developer community compared to competitors. |

---

## Recommended Deployment Model Strategy

The system utilizes a **Tiered Hybrid Model Routing Strategy** to optimize cost, latency, and capability:

```
                              +-------------------------+
                              | Ingested Security Alert |
                              +------------+------------+
                                           |
                                           v
                              +-------------------------+
                              |   Severity Assessment   |
                              +------------+------------+
                               /           |           \
                    Low/Medium /           | High/Crit  \ Local (Private)
                              /            |             \
                             v             v              v
               +-------------------+ +-------------+ +--------------------+
               | Gemini 1.5 Flash  | | Claude 3.5  | |   Llama 3 (70B)    |
               | - High speed      | | Sonnet      | | - Local GPU        |
               | - Lower cost      | | - Deep logic| | - Data privacy     |
               | - Initial triage  | | - Complex   | | - Air-gapped setup |
               |                   | |   scripts   | |                    |
               +-------------------+ +-------------+ +--------------------+
```

### 1. Tier 1: Real-Time Ingest Triaging (Gemini 1.5 Flash)
* **Use Case**: Runs on all incoming security logs that match low-to-medium severity alerts.
* **Why**: The low price and large context window allow the system to ingest massive logs without running up heavy API fees, filtering out false positives and summarizing event histories in seconds.

### 2. Tier 2: Deep Exploit Investigation & Remediation (Claude 3.5 Sonnet / Gemini 1.5 Pro)
* **Use Case**: Automatically triggered for alerts classified as High or Critical (e.g., data exfiltration indicators, administrative role modifications, active privilege escalation).
* **Why**: High-severity alerts require precise reasoning to formulate remediation scripts. Claude 3.5 Sonnet excels at generating valid, syntactically clean IaC modifications (Terraform/CloudFormation) and CLI sequences.

### 3. Tier 3: Local Offline Security Option (Llama 3 70B via vLLM)
* **Use Case**: Reserved for enterprise clients who operate under strict regulatory conditions (e.g., defense contractors, healthcare networks, financial institutions) that block transmitting system metadata or logs to external third-party API providers.
* **Why**: Allows the assistant to run completely air-gapped within the corporate VPC network on local server hardware.
