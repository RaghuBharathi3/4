# 22. Database Design

## 1. Database Technology Comparison

To manage normalized alert tables, user sessions, chat logs, and vector indexes, the assistant relies on a dual-database architecture: a relational/document engine for metadata and transactional integrity, and a vector engine for RAG guidelines.

| Database Option | Model Type | JSON Handling | Transaction Integrity (ACID) | Vector Search Capability | Primary Advantage | Primary Disadvantage |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **PostgreSQL** | Relational (SQL) | Excellent (native `JSONB` columns, indexes) | Full (Strict ACID compliance) | Excellent (via `pgvector` plugin) | Robust transactional support; query joining; JSON index features; pgvector supports combined SQL + vector operations. | Requires schema migrations; slightly more setup compared to flat file JSON stores. |
| **MongoDB** | Document (NoSQL)| Native BSON (perfect for JSON logs) | Good (supports transactions) | Basic (Atlas Vector Search) | Dynamic schemas allow storing raw logs of arbitrary formats without structure changes. | Weak query joins; higher memory footprint for complex transactional data. |
| **MySQL** | Relational (SQL) | Moderate (supports JSON type) | Full (Strict ACID compliance) | Poor (lacks native high-speed index plugins) | High read speed; standard query syntax; wide hosting availability. | Poor performance on JSON queries compared to PostgreSQL `JSONB`. |
| **Firebase** | Document NoSQL / Real-time | Good (JSON-like structure) | Basic | None | Extremely fast setup; native WebSockets synchronizations for real-time dashboards. | Strict document boundaries; expensive scaling; vendor lock-in. |

**Selection Recommendation**: **PostgreSQL** is selected as the primary metadata and transaction store due to its strict ACID compliance, robust `JSONB` column features (allowing sanitization caches of raw logs), and ability to store vectors directly in SQL tables via `pgvector` if a unified database model is preferred.

---

## 2. Transactional Database Schema Design

Below is the conceptual entity schema design (written as SQL DDL definitions for PostgreSQL):

```sql
-- Enums for state control
CREATE TYPE user_role AS ENUM ('security_admin', 'read_only_analyst');
CREATE TYPE alert_status AS ENUM ('PENDING_ANALYSIS', 'ANALYZING', 'RESOLVED');
CREATE TYPE message_role AS ENUM ('user', 'assistant');

-- 1. Users Table
CREATE TABLE users (
    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role user_role DEFAULT 'read_only_analyst',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 2. Incidents (Alerts) Table
CREATE TABLE incidents (
    incident_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_id VARCHAR(64) UNIQUE NOT NULL, -- SHA-256 hash of raw event
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    cloud_provider VARCHAR(20) NOT NULL, -- AWS, AZURE, GCP
    account_id VARCHAR(50) NOT NULL,
    region VARCHAR(50) NOT NULL,
    actor_identity VARCHAR(255) NOT NULL,
    source_ip VARCHAR(45) NOT NULL,
    target_resource_arn TEXT NOT NULL,
    event_name VARCHAR(100) NOT NULL,
    severity VARCHAR(20) NOT NULL, -- CRITICAL, HIGH, MEDIUM, LOW
    status alert_status DEFAULT 'PENDING_ANALYSIS',
    raw_payload_sanitized JSONB NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 3. AI Analysis Records Table
CREATE TABLE ai_analysis_records (
    analysis_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    incident_id UUID REFERENCES incidents(incident_id) ON DELETE CASCADE,
    executive_summary TEXT NOT NULL,
    exploit_walkthrough TEXT NOT NULL,
    remediation_script TEXT, -- Safe CLI blocks
    iaac_code TEXT, -- Terraform block suggestion
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 4. Chat Sessions Table
CREATE TABLE chat_sessions (
    session_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    title VARCHAR(255) DEFAULT 'New Chat Session',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 5. Chat Messages Table
CREATE TABLE chat_messages (
    message_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id UUID REFERENCES chat_sessions(session_id) ON DELETE CASCADE,
    role message_role NOT NULL,
    content TEXT NOT NULL,
    citations JSONB, -- Array of source hyperlinks used in RAG
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 6. Tamper-proof System Audit Table (WORM Target)
CREATE TABLE system_audit_logs (
    audit_id BIGSERIAL PRIMARY KEY,
    timestamp TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    user_id UUID, -- NULL for system actions
    action_performed VARCHAR(255) NOT NULL,
    resource_affected VARCHAR(255),
    ip_address VARCHAR(45) NOT NULL,
    status VARCHAR(50) NOT NULL
);
```

---

## 3. Vector Database Index Structure

When utilizing a dedicated vector database (like ChromaDB or Pinecone), document chunks are stored with the following vector schema:

### Vector Data Definitions
* **Dimension Count**: **1536** (matches standard `text-embedding-3-small` dimensions) or **768** (for Google `text-embedding-004`).
* **Distance Metric**: **Cosine Distance** ($1 - \text{Cosine Similarity}$).
* **Index Method**: Hierarchical Navigable Small World (HNSW) to support fast search queries:
  * `M`: 16 (max outgoing links per node in graph).
  * `efConstruction`: 200 (exploration depth during index building).

### Query Input mapping
When querying the Vector database, the system filters the search space by cloud provider and service before performing vector comparisons:

```
+-----------------------------------------------------------+
| Search Query: "Revoke security group ingress SSH port 22" |
+----------------------------+------------------------------+
                             |
                             v
+----------------------------+------------------------------+
| Metadata Filter:                                          |
| { "cloud_provider": "AWS", "service": "EC2" }             |
+----------------------------+------------------------------+
                             |
                             v
+----------------------------+------------------------------+
| Execute HNSW Vector search inside filtered index space    |
+-----------------------------------------------------------+
```
