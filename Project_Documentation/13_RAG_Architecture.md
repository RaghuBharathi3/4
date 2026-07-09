# 13. RAG Architecture

## Introduction

Retrieval-Augmented Generation (RAG) is a core component of the **Generative AI-Powered Cloud Security Assistant**. By querying external knowledge bases and security standards before invoking the LLM, the RAG architecture grounds the assistant in factual vendor guidelines, CIS benchmarks, and CVE databases, minimizing hallucinations and ensuring valid remediation advice.

---

## RAG Pipeline Flow

The diagram below details the ingestion and retrieval processes within the RAG framework:

```
================================================================================
                    1. DOCUMENT INGESTION PIPELINE (OFFLINE)
================================================================================
  +------------------+      +-------------------+      +--------------------+
  | CIS Security     |      | Cloud Vendor Docs |      | Threat Databases   |
  | Benchmarks (PDF) |      | (AWS/Azure/GCP)   |      | (CVE, MITRE ATT&CK)|
  +--------+---------+      +---------+---------+      +---------+----------+
           |                          |                          |
           +--------------------------+--------------------------+
                                      |
                                      v
                        +----------------------------+
                        | Recursive Character Parser |
                        | (512 token chunks / overlap)|
                        +-------------+--------------+
                                      |
                                      v
                        +----------------------------+
                        | Embedding Model API        |
                        | (text-embedding-3-small)   |
                        +-------------+--------------+
                                      |
                                      v
                        +----------------------------+
                        |  Vector DB Store           |
                        |  (ChromaDB / FAISS / Pin)  |
                        +----------------------------+

================================================================================
                    2. RETRIEVAL & GROUNDING PIPELINE (ONLINE)
================================================================================
  +----------------------+
  | Normalized Log Event |
  +----------+-----------+
             |
             v
  +----------------------+      +----------------------+
  | Key Terms Extraction |----->| Embedding Generation |
  +----------------------+      +----------+-----------+
                                           |
                                           v
  +----------------------+      +----------------------+
  | Vector DB Search     |<-----| Semantic query +     |
  | (Cosine Similarity)  |      | Metadata filter      |
  +----------+-----------+      +----------------------+
             |
             v
  +----------------------+
  | Cross-Encoder        |
  | Reranking            |
  +----------+-----------+
             | (Top 3 Chunks)
             v
  +----------------------+
  | LLM Prompt Grounding |
  +----------------------+
```

---

## Vector Database Comparison

The following table evaluates the vector databases considered for the RAG architecture:

| Vector Database Options | Deployment Model | Licensing | Scaling Capability | Primary Advantage | Primary Disadvantage |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Pinecone** | Cloud SaaS (Fully Managed) | Commercial | High (auto-scaling) | No operational overhead; low query latency; highly optimized indexing. | Expensive; requires internet access (not suitable for air-gapped environments). |
| **ChromaDB** | Local / Embedded | Open-Source (Apache 2.0) | Small to Medium | Exceptionally simple setup; runs in-process; ideal for prototypes and small deployments. | Limited performance when scaling beyond millions of vectors. |
| **FAISS** | Local Library (Facebook Research) | Open-Source (MIT) | High (GPU optimized) | Extremely fast CPU/GPU-based similarity search index calculations. | Lacks native document storage; requires writing custom index management scripts. |
| **Weaviate** | Self-Hosted / SaaS | Open-Source (BSD) | High (Distributed) | Supports hybrid search (keyword + vector); includes GraphQL APIs. | Higher deployment and memory overhead. |
| **Milvus** | Self-Hosted / Cloud | Open-Source (Apache 2.0) | Enterprise Grade | Built for multi-billion vector scale; cloud-native clustering architecture. | High infrastructure complexity (requires Kubernetes, MinIO, Pulsar). |

---

## Indexing & Chunking Strategy

To optimize retrieval precision, documents are processed through the following pipeline:

### Chunking Configuration
* **Chunking Method**: Recursive Character Text Splitter, segmenting documents by header hierarchies (`#`, `##`, `###`) to keep sections coherent.
* **Token Length**: **512 tokens**. This size fits neatly into grounding windows without diluting local semantic density.
* **Overlap Window**: **52 tokens** (approx. 10%) to prevent context loss at chunk boundaries.

### Metadata Tagging Scheme
Every chunk is indexed along with a structured metadata payload. This enables the retriever to filter the search space dynamically:

```json
{
  "chunk_id": "aws_iam_cis_5.1_01",
  "cloud_provider": "AWS",
  "service": "IAM",
  "topic": "SessionSecurity",
  "compliance_standard": "CIS_AWS_Benchmark_v1.4",
  "source_file": "docs/aws/securing_iam_v1.md",
  "last_updated": "2026-05-12T00:00:00Z"
}
```

---

## Retrieval & Reranking Workflow

1. **Metadata Filtering**: When a log alert arrives (e.g., from AWS GuardDuty), the orchestrator identifies the cloud provider (`AWS`) and the service (`IAM`). It applies a metadata filter to the Vector DB search, limiting queries to matching chunks.
2. **Semantic Search**: The system converts the security event summary into a query vector, searching the filtered database using Cosine Similarity.
3. **Cross-Encoder Reranking**: The vector database returns the top 10 chunks. The system then processes these through a local Cross-Encoder model (e.g., `ms-marco-MiniLM-L-6-v2`) to re-score the relevance of each chunk relative to the raw log, passing only the top 3 most relevant chunks to the LLM prompt.
