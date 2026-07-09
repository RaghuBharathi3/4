# Generative AI-Powered Cloud Security Assistant

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![PRs-Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](.github/pull_request_template.md)
[![Documentation](https://img.shields.io/badge/Docs-Complete-blue.svg)](#documentation-index)
[![UN SDG Alignment](https://img.shields.io/badge/UN_SDGs-Goal_4_%26_Goal_9-orange.svg)](Project_Documentation/29_SDG_Mapping.md)

An advanced generative AI solution designed to ingest JSON security telemetry from multi-cloud environments (AWS GuardDuty, AWS CloudTrail, Azure Defender, and GCP Security Command Center), sanitize data to protect privacy (PII scrubbing), reference safety/regulatory guidelines via a local vector database (RAG), and generate plain-language threat summaries along with executable remediation scripts.

---

## Table of Contents
- [Project Overview](#project-overview)
- [Key Objectives](#key-objectives)
- [High-Level Architecture](#high-level-architecture)
- [Technology Stack](#technology-stack)
- [Documentation Index](#documentation-index)
- [References](#references)
- [License](#license)

---

## Project Overview

In modern multi-cloud architectures, security teams are overwhelmed by the sheer volume of heterogeneous JSON security alerts. This **alert fatigue**, combined with a global deficit in cloud security expertise, often delays incident response. 

The **Generative AI-Powered Cloud Security Assistant** addresses this challenge by:
1. **Normalizing Logs**: Standardizing heterogeneous alerts from multiple cloud environments.
2. **Redacting PII**: Automatically scrubbing sensitive user data and credentials before sending prompts to external APIs.
3. **Retrieval-Augmented Generation (RAG)**: Anchoring LLM recommendations to trusted frameworks (like CIS Benchmarks).
4. **Actionable Explanations**: Presenting clear threat summaries and copyable CLI/IaC commands for remediation.

---

## Key Objectives

* **Log Ingestion & Normalization**: Standardize heterogeneous JSON alerts into a unified system schema.
* **Natural Language Explanation**: Convert cryptic logs into plain-language threat summaries.
* **Actionable Remediation**: Suggest verified, target-specific CLI commands and Infrastructure as Code adjustments.
* **Grounded RAG Search**: Reference responses directly against official guidelines to prevent hallucinations.
* **Security & Privacy Guardrails**: Scrub credentials and PII from prompts and enforce a Human-in-the-Loop review process.
* **SDG Target Alignment**: Support Goal 4 (Quality Education) and Goal 9 (Industry, Innovation, and Infrastructure).

---

## High-Level Architecture

The system utilizes a decoupled microservices design:
1. **Ingress API Node**: Consumes webhooks and buffers events using Redis.
2. **PII Redactor Worker**: Normalizes log fields and redacts secrets and IPs.
3. **Retrieval-Augmented Vector Engine**: Queries a vector index of security standards (like CIS Benchmarks) using semantic search.
4. **AI Inference Core**: Formulates system prompts, calls LLMs, and validates code block outputs.
5. **Interactive Console UI**: Displays alert timelines, interactive chat panels, and report compilers.

---

## Technology Stack

* **Languages**: Python (Backend & AI Orchestration), TypeScript (Frontend Interface).
* **Frameworks**: FastAPI (Backend API Router), Next.js & Tailwind CSS (Frontend Dashboard).
* **AI Toolsets**: LangChain (LLM Session Logic), LlamaIndex (RAG Ingestion), CrewAI (Multi-Agent Coordination).
* **Databases**: PostgreSQL (Transactional Metadata and Logs), ChromaDB (Local Vector Indexing).
* **DevOps**: Docker, Kubernetes, Prometheus, Grafana, OpenTelemetry.

---

## Documentation Index

This repository contains a comprehensive 32-part engineering documentation package under the [`Project_Documentation/`](Project_Documentation/) folder. Below is the hyperlinked index of all documents:

| Step | Section | Description |
|---|---|---|
| 01 | [Project Overview](Project_Documentation/01_Project_Overview.md) | Executive summary and conceptual definitions. |
| 02 | [Problem Statement](Project_Documentation/02_Problem_Statement.md) | Analysis of current cloud security operational challenges. |
| 03 | [Project Objectives](Project_Documentation/03_Project_Objectives.md) | Primary goals, target metrics, and educational outcomes. |
| 04 | [Project Scope](Project_Documentation/04_Project_Scope.md) | Functional boundaries, inclusions, and exclusions. |
| 05 | [Functional Requirements](Project_Documentation/05_Functional_Requirements.md) | MoSCoW-prioritized list of system capabilities. |
| 06 | [Non-Functional Requirements](Project_Documentation/06_Non_Functional_Requirements.md) | Operational constraints (security, latency, scaling). |
| 07 | [System Architecture](Project_Documentation/07_System_Architecture.md) | Microservices architecture and components overview. |
| 08 | [High-Level Design](Project_Documentation/08_High_Level_Design.md) | Block designs mapping alert processing phases. |
| 09 | [Low-Level Design](Project_Documentation/09_Low_Level_Design.md) | Class schemas, RAG index sizes, and prompt templates. |
| 10 | [Modular Architecture](Project_Documentation/10_Modular_Architecture.md) | Detailed specifications for the 7 system modules. |
| 11 | [AI Architecture](Project_Documentation/11_AI_Architecture.md) | Prompt strategies, log compression, and hallucination checks. |
| 12 | [LLM Architecture](Project_Documentation/12_LLM_Architecture.md) | Comparative model analysis and tiered routing strategies. |
| 13 | [RAG Architecture](Project_Documentation/13_RAG_Architecture.md) | Ingestion pipelines, chunk dimensions, and vector indices. |
| 14 | [Security Architecture](Project_Documentation/14_Security_Architecture.md) | Least privilege access, PII redactor filters, and network isolation. |
| 15 | [Data Flow Diagram](Project_Documentation/15_Data_Flow.md) | Context Level 0 and Subsystem Level 1 DFDs. |
| 16 | [Sequence Diagrams](Project_Documentation/16_Sequence_Diagrams.md) | Message sequence flows for logs, chats, and reports. |
| 17 | [Component Diagrams](Project_Documentation/17_Component_Diagrams.md) | Grouping of system services and operational containers. |
| 18 | [Class Diagrams](Project_Documentation/18_Class_Diagrams.md) | UML static class diagrams, properties, and methods. |
| 19 | [Use Case Diagrams](Project_Documentation/19_Use_Case_Diagrams.md) | System boundaries and human/automated actor interfaces. |
| 20 | [Activity Diagrams](Project_Documentation/20_Activity_Diagrams.md) | Procedural workflows for log triaging and chat sessions. |
| 21 | [Deployment Architecture](Project_Documentation/21_Deployment_Architecture.md) | Topology detailing EKS nodes, VPC subnets, and WAF blocks. |
| 22 | [Database Design](Project_Documentation/22_Database_Design.md) | SQL DDL schemas and vector indexing settings. |
| 23 | [API Design](Project_Documentation/23_API_Design.md) | Conceptual REST schemas and Websocket stream structures. |
| 24 | [Frameworks and Tools](Project_Documentation/24_Frameworks_and_Tools.md) | Comparative framework evaluations (LangChain, LlamaIndex, CrewAI). |
| 25 | [Technology Stack](Project_Documentation/25_Technology_Stack.md) | Comparison table for code environments and tool selections. |
| 26 | [Development Roadmap](Project_Documentation/26_Development_Roadmap.md) | 18-week Gantt schedule and project delivery phases. |
| 27 | [Testing Strategy](Project_Documentation/27_Testing_Strategy.md) | Unit, integration, performance, and LLM output evaluation. |
| 28 | [Risk Assessment](Project_Documentation/28_Risk_Assessment.md) | Security risks, cost analysis, and mitigation steps. |
| 29 | [SDG Mapping](Project_Documentation/29_SDG_Mapping.md) | Concrete linkages mapping the system to UN targets. |
| 30 | [Future Enhancements](Project_Documentation/30_Future_Enhancements.md) | Proposed extensions and 20 academic research topics. |
| 31 | [Presentation Guide](Project_Documentation/31_Project_Presentation_Guide.md) | Presentation outline and anticipated Viva questions & answers. |
| 32 | [Demo Script](Project_Documentation/32_Project_Demo_Script.md) | Scenario walkthrough script for demo reviews. |

---

## References

1. **United Nations Sustainable Development Goals**: [SDG Goal 4 & Goal 9 Target Indicators](https://sdgs.un.org/goals).
2. **CIS Benchmarks**: [Center for Internet Security Cloud Configuration Standards](https://www.cisecurity.org/benchmark/amazon_web_services).
3. **MITRE ATT&CK Matrix**: [Threat Tactics and Techniques Documentation](https://attack.mitre.org/).
4. **LangChain Documentation**: [LLM Application Development Framework](https://python.langchain.com/docs/get_started/introduction).
5. **LlamaIndex Documentation**: [Data Ingestion Pipelines Guide](https://docs.llamaindex.ai/en/stable/).

---

## License

This design documentation is released under the [MIT License](LICENSE). You are free to modify, distribute, and implement the system design described within these materials.
