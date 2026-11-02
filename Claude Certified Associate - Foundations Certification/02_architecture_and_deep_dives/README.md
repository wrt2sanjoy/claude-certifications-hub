# 02. Architecture & Deep Dives

This module covers enterprise production architectures, scaling trade-offs, and advanced features tested in **Domain 4 (Workflow Integration and Solution Design)** and **Domain 5 (Configuration and Knowledge Management)** of the CCAO-F exam.

---

## ⚡ Anthropic Prompt Caching Architecture

Prompt Caching is one of Anthropic's most distinctive enterprise capabilities. It allows developers to cache frequently reused context (such as large codebases, reference documentation, or standard system instructions) in the model's memory.

```text
Request 1 (Initial Cache Write):
+-------------------------------------------------------+---------------------+
| Static Context: API Specs, Docs (>1024 Tokens)        | User Query 1        |
| Cost: 1.25x Normal Base Input Cost (Cache Write)      | Normal Token Cost   |
+-------------------------------------------------------+---------------------+
                                   |
                                   v  [5-Minute Sliding TTL Window]
Request 2 (Cache Hit):
+-------------------------------------------------------+---------------------+
| Static Context: API Specs, Docs (CACHE HIT!)          | User Query 2        |
| Cost: 0.10x Normal Base Input Cost (90% SAVINGS!)     | Normal Token Cost   |
+-------------------------------------------------------+---------------------+
```

```mermaid
sequenceDiagram
    autonumber
    actor Client as Application Client
    participant API as Anthropic Messages API
    participant Cache as KV Prompt Cache Store
    participant Claude as Claude Inference Engine

    Note over Client,Claude: Call 1: Cache Miss / Write (Creation)
    Client->>API: POST /v1/messages (System prompt + 50k tokens doc + cache_control: {"type": "ephemeral"})
    API->>Cache: Store Key-Value state (5-min TTL timer starts)
    API->>Claude: Run full inference
    Claude-->>Client: Response (Tokens billed: Cache Creation 1.25x rate)

    Note over Client,Claude: Call 2: Within 5 Minutes (Cache Hit)
    Client->>API: POST /v1/messages (Identical 50k tokens doc + New User Query)
    API->>Cache: Match hash prefix? YES (Cache Read)
    Cache-->>Claude: Load cached attention activations instantly
    API->>Claude: Process only new user tokens
    Claude-->>Client: Response (Tokens billed: Cache Read 0.10x rate - 90% discount! TTL resets to 5 min)
```

### Prompt Caching Canonical Rules & Constraints

1. **Prefix Matching Requirement**: The cached prefix must be identical byte-for-byte. Place **static** content at the top and **dynamic** content at the bottom.
2. **Minimum Token Thresholds**:
   - **Claude 3.5 Sonnet & Claude 3 Opus**: Minimum **1,024 tokens**.
   - **Claude 3.5 Haiku**: Minimum **2,048 tokens**.
   - Prompts below these thresholds cannot be cached and will trigger normal input billing.
3. **Sliding 5-Minute TTL**: Every cache hit resets the 5-minute expiration clock.
4. **Economic Break-Even**:
   - Cache Write = `1.25x` regular input token price.
   - Cache Read = `0.10x` regular input token price.
   - Break-even occurs on the **very second request**!

---

## 📂 Claude Projects: Enterprise Knowledge Architecture

**Claude Projects** (`claude.ai`) provide non-technical and enterprise teams with an autonomous, collaborative environment.

```text
+---------------------------------------------------------------------------------+
|                                 CLAUDE PROJECT                                  |
|                                                                                 |
|  +---------------------------------------------------------------------------+  |
|  | Project Custom Instructions (Applied automatically to every chat session) |  |
|  +---------------------------------------------------------------------------+  |
|                                                                                 |
|  +---------------------------------------------------------------------------+  |
|  | Project Knowledge Base (Up to 200,000 tokens of PDFs, Markdown, Code)     |  |
|  +---------------------------------------------------------------------------+  |
|                                      |                                          |
|            +-------------------------+-------------------------+                |
|            |                                                   |                |
|            v                                                   v                |
|   +-------------------+                               +-------------------+     |
|   |   Chat Thread A   |                               |   Chat Thread B   |     |
|   | (Member 1 Query)  |                               | (Member 2 Query)  |     |
|   +-------------------+                               +-------------------+     |
+---------------------------------------------------------------------------------+
```

```mermaid
graph TD
    Project["Claude Project<br/>(Team Workspace)"]
    Inst["Custom Instructions<br/>(Style, Rules, Role Persona)"]
    KB["Project Knowledge Base<br/>(Up to 200,000 tokens shared docs)"]
    Thread1["Chat Thread 1<br/>(Product Manager)"]
    Thread2["Chat Thread 2<br/>(Lead Architect)"]
    Artifact["Shared Artifacts<br/>(Code, Diagrams, PRDs)"]

    Project --> Inst
    Project --> KB
    Project --> Artifact
    Inst --> Thread1
    KB --> Thread1
    Inst --> Thread2
    KB --> Thread2
```

### When to Use Projects vs Artifacts vs Standard Chat

| Surface | Best Used For | Key Capabilities | Persistence & Sharing |
| :--- | :--- | :--- | :--- |
| **Standard Claude Chat** | Ad-hoc, one-off questions, exploratory brainstorming | Ephemeral context, quick turnaround | Private to user; no shared persistent knowledge |
| **Claude Projects** | Multi-document team initiatives, persistent domain guidelines | Pre-loaded 200k token knowledge base, custom project instructions | Shared across team workspace; persistent |
| **Claude Artifacts** | Standalone code files, interactive web apps, SVG graphics, PRDs | Side-by-side dedicated editor panel, instant code rendering | Downloadable, editable, shareable directly |
| **Messages API** | Production software, custom RAG pipelines, microservices | Headless automation, fine-grained temperature, prompt caching | Managed by developer application database |

---

## ⚖️ Architectural Trade-off: 200k Long Context vs Vector RAG

A frequent exam scenario contrasts Anthropic's **200,000-token native context window** with traditional **Vector RAG (Retrieval-Augmented Generation)**.

```text
+-------------------------------------------------------------------------------+
|                      200k Native Long-Context Window                          |
|  Pros: Zero indexing delay, perfect holistic document synthesis, no chunking  |
|  Cons: Cost per query is higher if uncached; capped at ~150k words            |
|  Best For: Legal contract review, single codebase audit, financial reports    |
+-------------------------------------------------------------------------------+
                                      vs
+-------------------------------------------------------------------------------+
|                        Vector RAG (Embeddings + Vector DB)                    |
|  Pros: Scales to millions of documents, retrieves only top-k chunks           |
|  Cons: Loss of global document context, chunk boundary truncation, retrieval  |
|  Best For: Multi-gigabyte enterprise knowledge bases, global help centers     |
+-------------------------------------------------------------------------------+
```

---

## ⚠️ Exam Trap Warnings for Domain 4 & 5

> [!WARNING]
> **Exam Trap 1: Prompt Caching Dynamic Placement**
> If a developer puts the changing user query *before* the static 50,000-word product manual, the cache will NEVER hit. Caching requires an identical byte-for-byte prefix from token 0.

> [!WARNING]
> **Exam Trap 2: Sub-Threshold Caching Attempts**
> Attempting to cache a 500-token system prompt on Claude 3.5 Sonnet will silently bypass caching because the minimum threshold is **1,024 tokens**.

> [!WARNING]
> **Exam Trap 3: Project Knowledge Limits**
> Claude Project knowledge base is bounded by the model's context window (~200,000 tokens total). Uploading a 20-volume encyclopedia exceeds project knowledge limits and requires a dedicated external RAG or filtering solution.
