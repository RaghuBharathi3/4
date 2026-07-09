# Generative AI-Powered Cloud Security Assistant

## Overview

The **Generative AI-Powered Cloud Security Assistant** is a solution designed to simplify cloud security operations using Large Language Models (LLMs) and Retrieval-Augmented Generation (RAG). The assistant ingests JSON alert telemetry from multi-cloud environments (AWS GuardDuty/CloudTrail, Azure Defender, and GCP Security Command Center), parses and sanitizes the logs to redact PII, queries a vector index of security standards (such as CIS benchmarks), and generates plain-language threat explanations and copyable remediation scripts.

This architecture acts as an educational and operational tool for administrators, accelerating threat response, reducing alert fatigue, and lowering the barrier to cloud security expertise.

---

## Objectives Summary
* **Log Ingestion & Normalization**: Standardize heterogeneous JSON alerts into a unified system schema.
* **Natural Language Explanation**: Convert cryptic logs into plain-language threat summaries.
* **Actionable Remediation**: Suggest verified, target-specific CLI commands and Infrastructure as Code adjustments.
* **Grounded RAG Search**: Reference responses directly against official guidelines to prevent hallucinations.
* **Security & Privacy Guardrails**: Scrub credentials and PII from prompts and enforce a Human-in-the-Loop review process.
* **SDG Target Alignment**: Support Goal 4 (Quality Education) and Goal 9 (Industry, Innovation, and Infrastructure).

---

## Architecture Summary
The system is built on a decoupled microservices model:
1. **Ingress API Node**: Consumes webhooks and handles Redis buffer scheduling.
2. **PII Redactor Worker**: Normalizes log fields and redacts secrets and IPs.
3. **Retrieval-Augmented Vector Engine**: Uses semantic search queries to retrieve relevant reference guidelines.
4. **AI Inference & Verification Core**: Hydrates system prompts, queries LLM endpoints, and validates code block outputs.
5. **Interactive Console UI**: Displays alert timelines, interactive chat panels, and report compilers.

---

## Technology Stack

* **Languages**: Python (Backend & AI Orchestration), TypeScript (Frontend Interface).
* **Frameworks**: FastAPI (Backend API Router), Next.js & Tailwind CSS (Frontend Dashboard).
* **AI Toolsets**: LangChain (LLM Session Logic), LlamaIndex (RAG Document Ingestion), CrewAI (Multi-Agent Coordination).
* **Databases**: PostgreSQL (Transactional Metadata and Logs), ChromaDB (Local Vector Indexing).
* **DevOps**: Docker, Kubernetes, Prometheus, Grafana, OpenTelemetry.

---

## Folder Directory Structure

This documentation package contains the following structure under the `Project_Documentation/` folder:

```
Project_Documentation/
├── 01_Project_Overview.md              <- Executive summary and conceptual definitions.
├── 02_Problem_Statement.md             <- Analysis of current cloud security operational challenges.
├── 03_Project_Objectives.md            <- Primary goals, target metrics, and educational outcomes.
├── 04_Project_Scope.md                 <- Functional boundaries, inclusions, and exclusions.
├── 05_Functional_Requirements.md       <- MoSCoW-prioritized list of system capabilities.
├── 06_Non_Functional_Requirements.md   <- Operational constraints (security, latency, scaling).
├── 07_System_Architecture.md           <- Microservices architecture and components overview.
├── 08_High_Level_Design.md             <- Block designs mapping alert processing phases.
├── 09_Low_Level_Design.md              <- Class schemas, RAG index sizes, and prompt templates.
├── 10_Modular_Architecture.md          <- Detailed specifications for the 7 system modules.
├── 11_AI_Architecture.md               <- Prompt strategies, log compression, and hallucination checks.
├── 12_LLM_Architecture.md               <- Comparative model analysis and tiered routing strategies.
├── 13_RAG_Architecture.md              <- Ingestion pipelines, chunk dimensions, and vector indices.
├── 14_Security_Architecture.md         <- Least privilege access, PII redactor filters, and network isolation.
├── 15_Data_Flow.md                     <- Context Level 0 and Subsystem Level 1 DFDs.
├── 16_Sequence_Diagrams.md             <- Message sequence flows for logs, chats, and reports.
├── 17_Component_Diagrams.md            <- Grouping of system services and operational containers.
├── 18_Class_Diagrams.md                <- UML static class diagrams, properties, and methods.
├── 19_Use_Case_Diagrams.md             <- System boundaries and human/automated actor interfaces.
├── 20_Activity_Diagrams.md             <- Procedural workflows for log triaging and chat sessions.
├── 21_Deployment_Architecture.md       <- Topology detailing EKS nodes, VPC subnets, and WAF blocks.
├── 22_Database_Design.md               <- SQL DDL schemas and vector indexing settings.
├── 23_API_Design.md                    <- Conceptual REST schemas and Websocket stream structures.
├── 24_Frameworks_and_Tools.md          <- Comparative framework evaluations (LangChain, LlamaIndex, CrewAI).
├── 25_Technology_Stack.md              <- Comparison table for code environments and tool selections.
├── 26_Development_Roadmap.md           <- 18-week Gantt schedule and project delivery phases.
├── 27_Testing_Strategy.md              <- Unit, integration, performance, and LLM output evaluation.
├── 28_Risk_Assessment.md               <- Security risks, cost analysis, and mitigation steps.
├── 29_SDG_Mapping.md                   <- Concrete linkages mapping the system to UN targets.
├── 30_Future_Enhancements.md           <- Proposed extensions and 20 academic research topics.
├── 31_Project_Presentation_Guide.md    <- Presentation outline and anticipated Viva questions & answers.
├── 32_Project_Demo_Script.md           <- Scenario walkthrough script for demo reviews.
└── README.md                           <- Index map guide (this file).
```

---

## References

1. **United Nations Sustainable Development Goals**: [SDG Goal 4 & Goal 9 Target Indicators](https://sdgs.un.org/goals).
2. **CIS Benchmarks**: [Center for Internet Security Cloud Configuration Standards](https://www.cisecurity.org/benchmark/amazon_web_services).
3. **MITRE ATT&CK Matrix**: [Threat Tactics and Techniques Documentation](https://attack.mitre.org/).
4. **LangChain Documentation**: [LLM Application Development Framework](https://python.langchain.com/docs/get_started/introduction).
5. **LlamaIndex Documentation**: [Data Indexing and Ingestion Pipelines Guide](https://docs.llamaindex.ai/en/stable/).

---

## License

This design documentation is released under the **MIT License**. You are free to modify, distribute, and implement the system design described within these materials.
