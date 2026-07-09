# 18. Class Diagrams

## Introduction

The UML class diagram describes the static structure of the **Generative AI-Powered Cloud Security Assistant**. It details classes, database interfaces, helper modules, attributes, methods, and relationships.

---

## System Class Diagram

The diagram below details the core classes and their operational dependencies:

```mermaid
classDiagram
    class AlertIngestionGateway {
        +string ipAddress
        +int port
        +receiveWebhook(request: JSON) HTTPResponse
        -validateSignature(request: JSON) bool
        -pushToQueue(rawLog: string) void
    }

    class LogSanitizer {
        +list regexFilters
        +redactPII(rawLog: string) string
        +normalizeSchema(cleanLog: string) NormalizedEvent
    }

    class NormalizedEvent {
        +string eventId
        +DateTime timestamp
        +string cloudProvider
        +string accountId
        +string region
        +string actorIdentity
        +string targetResourceArn
        +string eventName
        +string severity
        +dict rawPayload
    }

    class RelationalDBManager {
        +string connectionString
        +saveAlert(event: NormalizedEvent) bool
        +fetchAlert(eventId: string) NormalizedEvent
        +updateAlertStatus(eventId: string, status: string) bool
    }

    class VectorDBManager {
        +string indexName
        +querySimilarity(vector: list, filter: dict) list
        +insertChunk(chunk: DocumentChunk) bool
    }

    class AIOrchestrator {
        +LLMClient client
        +PromptBuilder promptBuilder
        +GuardrailsValidator guardrails
        +runTriage(event: NormalizedEvent) AnalysisOutput
        +streamChat(query: string, history: list) TokenStream
    }

    class PromptBuilder {
        +dict templates
        +buildTriagePrompt(event: NormalizedEvent, context: list) string
        +buildChatPrompt(query: string, context: list, history: list) string
    }

    class GuardrailsValidator {
        +list blocklist
        +verifyInbound(prompt: string) bool
        +verifyOutbound(rawOutput: string) bool
    }

    class ReportingEngine {
        +string outputDirectory
        +compilePostMortem(eventId: string) string
        -renderPDF(markdownContent: string) string
    }

    %% Associations & Dependencies
    AlertIngestionGateway ..> LogSanitizer : uses
    LogSanitizer ..> NormalizedEvent : creates
    AlertIngestionGateway ..> RelationalDBManager : saves
    AIOrchestrator --> PromptBuilder : aggregates
    AIOrchestrator --> GuardrailsValidator : delegates validation
    AIOrchestrator ..> VectorDBManager : queries docs
    AIOrchestrator ..> RelationalDBManager : updates status
    ReportingEngine ..> RelationalDBManager : queries metrics
```

---

## Class Definitions & Methods

### 1. `AlertIngestionGateway`
* **Attributes**:
  * `ipAddress`, `port`: Network configurations.
* **Methods**:
  * `receiveWebhook()`: Gateway entry point for POST events.
  * `validateSignature()`: Ensures incoming logs originate from verified cloud webhook channels.
  * `pushToQueue()`: Serializes and offloads logs to the Redis queue.

### 2. `LogSanitizer`
* **Attributes**:
  * `regexFilters`: Patterns matching secret keys, access tokens, email formats, and public IP networks.
* **Methods**:
  * `redactPII()`: Replaces sensitive indicators with generalized tokens.
  * `normalizeSchema()`: Maps parsed values into the unified `NormalizedEvent` format.

### 3. `AIOrchestrator`
* **Attributes**:
  * `client`: Concrete wrapper representing model APIs (Gemini, Claude).
  * `promptBuilder`: Reference to prompt construction module.
  * `guardrails`: Instance of safety verification shield.
* **Methods**:
  * `runTriage()`: Executes threat analysis pipelines for newly ingested events.
  * `streamChat()`: Formulates query prompts and streams generated text back to browser portals.

### 4. `GuardrailsValidator`
* **Attributes**:
  * `blocklist`: Prohibited terminal utilities and wildcard permission operations.
* **Methods**:
  * `verifyInbound()`: Scans prompts for adversarial override payloads.
  * `verifyOutbound()`: Scans LLM responses to ensure recommended scripts are syntactically valid and secure.

### 5. `ReportingEngine`
* **Attributes**:
  * `outputDirectory`: Storage path for compiled PDF reports.
* **Methods**:
  * `compilePostMortem()`: Builds a markdown incident report by joining alert histories and AI analyses.
  * `renderPDF()`: Converts compiled markdown files into page-numbered PDF documents.
