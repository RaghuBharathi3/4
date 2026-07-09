# 29. SDG Mapping

## Introduction

The **Generative AI-Powered Cloud Security Assistant** aligns with two United Nations Sustainable Development Goals (SDGs): **SDG 4 (Quality Education)** and **SDG 9 (Industry, Innovation, and Infrastructure)**. This document explains how the project's features support these goals.

---

## Detailed Mapping Matrix

```
                          +-------------------------------+
                          |    Sustainable Development    |
                          |          Goals (SDGs)         |
                          +---------------+---------------+
                                          |
                  +-----------------------+-----------------------+
                  |                                               |
                  v                                               v
      +-----------------------+                       +-----------------------+
      |  SDG 4: Quality Ed.   |                       |  SDG 9: Industry &    |
      |                       |                       |  Infrastructure       |
      |  Target 4.4: Uplift   |                       |  Target 9.c: Access   |
      |  technical skills     |                       |  to ICT infrastructure|
      +-----------+-----------+                       +-----------+-----------+
                  |                                               |
                  v (Implementations)                             v (Implementations)
      +-----------------------+                       +-----------------------+
      | - Explanatory Triage  |                       | - Open-source RAG SIEM|
      | - MITRE & CVE Guides  |                       | - Accelerating MTTR   |
      | - Contextual Learning |                       | - Affordable SecOps   |
      +-----------------------+                       +-----------------------+
```

---

## SDG 4: Quality Education

### Target 4.4
> *"By 2030, substantially increase the number of youth and adults who have relevant skills, including technical and vocational skills, for employment, decent jobs and entrepreneurship."*

### How the Project Contributes
Cloud security is often seen as highly complex, requiring years of experience to interpret raw log events. The assistant makes cloud security more accessible:

1. **Translating Raw Technical Telemetry into Natural Language**: By converting cryptic JSON logs into readable explanations, the assistant acts as a mentor for junior admins and students, helping them learn how threats operate.
2. **Contextual Security Education**: Every analyzed incident includes an "Educational Deep Dive" section explaining the underlying mechanics (e.g., how an SSRF exploit can lead to compromised metadata credentials).
3. **Traceable Knowledge Citations**: Linking recommendations directly to official documentation (e.g., CIS benchmarks, AWS guidelines) encourages users to reference authoritative security sources.

---

## SDG 9: Industry, Innovation, and Infrastructure

### Target 9.c
> *"Significantly increase access to information and communications technology and strive to provide universal and affordable access to the Internet..."*

### How the Project Contributes
As businesses digitize, secure cloud infrastructure becomes essential for operational resilience. The assistant helps organizations secure this infrastructure:

1. **Improving Cybersecurity Resilience**: Minimizing threat response times helps protect vital digital systems, preventing downtime and data exposure.
2. **Making Security Tools More Affordable**: Building the assistant on open-source frameworks (FastAPI, LlamaIndex, ChromaDB) provides small-and-medium enterprises (SMEs) and public organizations with access to advanced security analysis tools without high licensing fees.
3. **Bridging the Cybersecurity Skill Deficit**: Junior IT staff can triage and mitigate cloud misconfigurations, reducing the need for expensive dedicated security teams.
