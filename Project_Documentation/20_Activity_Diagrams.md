# 20. Activity Diagrams

## Introduction

Activity diagrams depict the procedural workflow and decision pathways within the **Generative AI-Powered Cloud Security Assistant**. They highlight routing choices, security validations, and fallback states.

---

## 1. Automated Log Triaging & Analysis Flow

This diagram details the sequence of checks and processing tasks executed upon log ingestion:

```mermaid
graph TD
    Start([Log Event Received]) --> SigCheck{Is Webhook <br> Signature Valid?}
    
    SigCheck -- No --> Reject[Drop Connection / Log Warn] --> End([Process Terminated])
    SigCheck -- Yes --> Queue[Push Alert to Redis Queue]
    
    Queue --> Pop[Parse & Redact PII Payload]
    Pop --> Normalize[Map Event to Internal Schema]
    
    Normalize --> QueryVDB[Vector Database Search]
    QueryVDB --> BuildPrompt[Compile Grounded System Prompt]
    
    BuildPrompt --> LLM[Execute LLM API Call]
    LLM --> Guard{Does Output <br> Pass Guardrails?}
    
    Guard -- No --> Recorrect[Trigger Local Rule-Based Plan]
    Guard -- Yes --> Stream[WebSocket Broadcast to Dashboard]
    
    Recorrect --> Stream
    Stream --> SaveDB[Update PostgreSQL status: RESOLVED]
    SaveDB --> Audit[Write Action Event to Audit WORM Store]
    Audit --> Complete([Alert Analysis Complete])
```

---

## 2. Interactive conversational Query Flow

This workflow illustrates how the assistant processes user questions about security events:

```mermaid
graph TD
    QueryStart([User Inputs Chat Question]) --> InjectionShield{Is Prompt <br> Injection Detected?}
    
    InjectionShield -- Yes --> Refuse[Render Policy Warning Block] --> QueryEnd([Query Handled])
    InjectionShield -- No --> GetHistory[Fetch Conversation History Store]
    
    GetHistory --> RAGSearch[Execute Context Search in Vector DB]
    RAGSearch --> InjectContext[Inject Relevant Documents to Prompt]
    
    InjectContext --> CallLLM[Inference Call to LLM API]
    CallLLM --> ExtractCode{Are Script Blocks <br> Present in Output?}
    
    ExtractCode -- No --> RenderText[Stream Markdown Response to UI]
    ExtractCode -- Yes --> ValidateCode{Does Code Pass <br> Safety Rules?}
    
    ValidateCode -- No --> RedactCode[Remove Code Block / Inject Safe CLI Pattern] --> RenderText
    ValidateCode -- Yes --> RenderText
    
    RenderText --> SaveChat[Save Chat Message to PostgreSQL History]
    SaveChat --> QueryEnd
```
