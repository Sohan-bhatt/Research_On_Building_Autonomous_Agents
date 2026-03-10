# Memory Layer — Storage Approaches & Retrieval Strategies
> Study notes on how Mem0, Zep, and Letta actually store and retrieve memory, and how to design your own.

---

## The Big Picture

There are **3 core storage paradigms** used by memory companies. Each answers a different question:

| Paradigm | Core Question It Answers | Used By |
|---|---|---|
| Vector DB | *"What is semantically similar to this query?"* | Mem0 |
| Knowledge Graph | *"How are entities related, and did that change over time?"* | Zep / Graphiti |
| Relational / JSON | *"What structured facts do I know about this agent/user?"* | Letta |

Production systems almost always combine 2 or more of these.

---

## 1. Vector Database Storage (Mem0's Approach)

### What it does
Converts memory into a numerical embedding (a vector of ~1536 numbers) and stores it. Retrieval works by finding vectors closest to the query vector — **semantic similarity search**.

### Memory Record Structure
```json
{
  "id": "uuid",
  "user_id": "user123",
  "content": "User prefers dark mode",
  "embedding": [0.12, -0.45, 0.78, "...1536 dims"],
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

### How Mem0 Works Internally
```
Raw conversation
      │
      ▼
LLM extracts facts
("User prefers dark mode", "User is a developer")
      │
      ▼
Each fact is embedded → vector
      │
      ▼
Stored in Qdrant (open source) or PostgreSQL + pgvector
      │
      ▼
Retrieval: embed the query → cosine similarity → top-K facts returned
```

### Databases Used
Qdrant · Pinecone · Weaviate · pgvector (PostgreSQL extension)

### Trade-offs
| ✅ Strengths | ❌ Weaknesses |
|---|---|
| Fast retrieval at scale | No understanding of relationships between facts |
| Captures semantic meaning (not just keywords) | Can't track how facts change over time |
| Easy to set up with pgvector | Similarity ≠ correctness (hallucinated proximity) |

---

## 2. Knowledge Graph Storage (Zep / Graphiti's Approach)

### What it does
Stores memory as a **graph of entities (nodes) connected by relationships (edges)**. The key innovation in Graphiti is **temporal intelligence** — relationships have validity timestamps, so the graph knows when facts were true, not just what was true.

### Data Model
```
Nodes (Entities):
  Person  → "Robbie"
  Brand   → "Adidas", "Nike"
  Event   → "shoes_fell_apart"

Edges (Relationships):
  Robbie ──[owns]──────────► Adidas shoes
  Robbie ──[switched_to]───► Nike
  Adidas ──[experienced]───► fell_apart
```

### Temporal Intelligence (Graphiti's Core Innovation)
```
"Robbie likes Adidas"  valid_from: 2024-01-01  valid_until: 2024-10-14
"Robbie likes Nike"    valid_from: 2024-10-14  valid_until: NULL (still true)
```

This is critical. Without temporal tracking, a memory system will confidently tell you outdated facts. A vector DB storing "Robbie likes Adidas" has no idea that's no longer true.

### Databases Used
Neo4j · FalkorDB

### Trade-offs
| ✅ Strengths | ❌ Weaknesses |
|---|---|
| Understands relationships between entities | Complex to build and maintain |
| Tracks how facts change over time | Slower retrieval than vector search |
| Ideal for org-level modeling (who owns what, who approved what) | Requires structured entity extraction |

> **This is the paradigm GeniOS is built on.** The Organisational Context Graph is a knowledge graph that models entities, relationships, authority, and live operational state.

---

## 3. Relational / JSON Storage (Letta's Approach)

### What it does
Stores memory as **structured blocks in PostgreSQL tables**. Each agent has defined memory blocks (persona, human profile, facts) with character limits — forcing the system to prioritise what stays in context.

### Database Schema
```sql
agents        (id, name, created_at, ...)
memory_blocks (id, agent_id, block_type, content, limit)
messages      (id, agent_id, role, content, timestamp)
tools         (id, agent_id, name, schema, ...)
```

### Memory Blocks (Letta's Abstraction)
```xml
<memory_blocks>
  <persona>
    <value>I am a helpful assistant...</value>
  </persona>
  <human>
    <value>User name: John, prefers code examples</value>
  </human>
  <facts>
    <value>
      - User is a Python developer
      - Prefers concise responses
    </value>
  </facts>
</memory_blocks>
```

### Trade-offs
| ✅ Strengths | ❌ Weaknesses |
|---|---|
| Simple, reliable, ACID-compliant | No semantic search (exact match only) |
| Easy to inspect and debug | Doesn't scale well for large unstructured memory |
| Great for structured agent state | Requires pre-defined schema — less flexible |

---

## 4. Retrieval Strategies

How you store memory determines how you retrieve it. Each strategy has a different strength:

| Strategy | How It Works | Best For |
|---|---|---|
| **Vector Similarity** | Embed query → find nearest neighbor vectors | Semantically fuzzy recall ("something about the user's preferences") |
| **Keyword / BM25** | Exact text matching with term frequency ranking | Precise recall of specific terms, names, codes |
| **Graph Traversal** | Follow relationship edges from a starting node | Understanding connections ("who approved this?", "what changed?") |
| **SQL Exact Match** | Primary key or column lookup | Structured facts, IDs, known attributes |
| **Hybrid** | Combine 2+ of the above with score fusion | Production systems that need both precision and recall |

---

## 5. Hybrid Architecture — Production Best Practice

Most real systems run all three retrieval strategies in parallel and fuse the scores:

```
Query Input
     │
     ▼
┌─────────────┐
│ Orchestrator│
└──────┬──────┘
       │
  ┌────┴────────────────┐
  │         │           │
  ▼         ▼           ▼
Vector    Graph      Keyword
Search  Traversal    (BM25)
  │         │           │
  └────┬────┘           │
       └────────┬────────┘
                ▼
         Score Fusion
         (rerank + combine)
                │
                ▼
         Top-K Results
         → injected into
           LLM context
```

### Example Score Fusion Formula
```
final_score = (0.5 × vector_score) + (0.3 × graph_score) + (0.2 × keyword_score)
```

The weights are tunable based on your use case. For an org context system like GeniOS, you'd weight graph traversal higher because relationships matter more than semantic similarity.

---

## 6. Key Design Decisions (What to Actually Use)

| Decision | Recommendation | Why |
|---|---|---|
| **Just starting out** | PostgreSQL + pgvector | Simple, good enough, one database to manage |
| **Need relationships** | Add Neo4j or use Postgres graph extensions | Entity relationships can't be modelled in flat vectors |
| **Need temporal tracking** | Add `valid_from` / `valid_until` timestamps | Know when facts were true, not just what was true |
| **Need scale** | Move to dedicated Qdrant or Pinecone | PostgreSQL vector search degrades at very high volume |
| **Production system** | Hybrid (vector + graph + keyword) | No single strategy covers all retrieval needs |

---

## 7. What Each Company Uses (Summary)

```
Mem0       → Qdrant (vector) + PostgreSQL
Zep        → Neo4j (knowledge graph) + Graphiti temporal layer
Letta      → PostgreSQL / Aurora (relational memory blocks)

GeniOS     → Neo4j (org graph) + PostgreSQL (structured facts)
             + Qdrant (semantic search) + Redis (working memory)
             [Hybrid — all four, each serving a different memory tier]
```

---

## Quick Mental Model

```
"What happened before?"         → Vector DB (Mem0 style)
"How are things related?"       → Knowledge Graph (Zep style)
"What structured facts exist?"  → Relational DB (Letta style)
"What is the org's live state?" → Context Graph (GeniOS)
```

The more complex your agents and the more they need to coordinate with each other, the further right on this spectrum you need to go.

---

*Notes compiled from memory architecture research — March 2026*
