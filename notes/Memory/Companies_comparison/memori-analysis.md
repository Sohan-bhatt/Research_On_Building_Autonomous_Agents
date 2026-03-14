# MEMORI (MEMORI LABS) - COMPLETE ANALYSIS

---

## 1. COMPANY OVERVIEW

| Attribute | Details |
|-----------|---------|
| **Company Name** | Memori Labs (GibsonAI Inc) |
| **Founded** | 2024 |
| **Headquarters** | New York, USA |
| **Funding** | $3.5M Seed (May 2024) |
| **Product** | Memori Cloud - SQL-native memory layer |
| **Employees** | 8 (+22% YoY) |
| **Website** | memorilabs.ai |
| **GitHub** | github.com/MemoriLabs |

---

## 2. WHAT MEMORI DOES

Memori is a **persistent memory layer for AI agents** that enables:

- **Conversation memory**: Remembers past interactions across sessions
- **Preference recall**: Stores user preferences and facts
- **Cost reduction**: Claims >60% reduction in inference costs by avoiding repeated context injection
- **Multi-agent support**: Specialized agents for memory processing

---

## 3. MEMORI ARCHITECTURE

### Core Components

```
┌─────────────────────────────────────────────────────────────┐
│                      MEMORI SYSTEM                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   │
│  │  Memory     │    │  Conscious  │    │   Memory    │   │
│  │  Agent      │    │  Agent      │    │   Search    │   │
│  │             │    │             │    │   Engine    │   │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘   │
│         │                  │                  │             │
│         └──────────────────┼──────────────────┘             │
│                            │                                │
│                    ┌───────┴───────┐                       │
│                    │  SQL Database │                       │
│                    │  (PostgreSQL) │                       │
│                    └───────────────┘                       │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              API Layer (REST)                        │   │
│  │  - /store (save memory)                             │   │
│  │  - /recall (retrieve memory)                        │   │
│  │  - /search (semantic search)                         │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Memory Agent
- Processes every conversation with structured outputs using Pydantic models
- Extracts facts, entities, and key information
- Stores in SQL database with proper schema

### Conscious Agent
- Copies "conscious-info" labeled memories to short-term memory
- Handles immediate context needs
- Manages what's relevant for current session

### Memory Search Engine
- Intelligently retrieves and injects relevant context
- Based on user queries
- Semantic search capabilities

### Dual Memory Modes

#### Conscious Ingest Mode (`conscious_ingest=True`)
- One-shot context injection at conversation start
- Essential memory promotion to short-term memory
- Runs once at program startup
- Use case: Persistent context for entire conversation sessions

#### Standard Mode
- On-demand memory retrieval
- Lazy loading of context
- Use case: Efficient resource usage

---

## 4. HOW IT WORKS (STEP BY STEP)

### Step 1: User/Agent Sends Query
```
User: "What's my name?"
```

### Step 2: Memory Search Engine Retrieves Context
```sql
SELECT * FROM memories 
WHERE user_id = ? 
ORDER BY importance DESC 
LIMIT 5
```

### Step 3: Context Injected into LLM Prompt
```
System: You are helpful.
User name: John (from memory)
User: What's my name?
```

### Step 4: LLM Generates Response
```
LLM: Your name is John!
```

### Step 5: Memory Agent Processes Conversation
```
- Extract facts: {name: "John"}
- Extract preferences: {}
- Store in SQL with embeddings
```

### Step 6: Next Session - Memory Persists
```
User: "Hi"
System: Injects John's name automatically
```

---

## 5. DATA MODEL

### Memory Structure (Pydantic)
```python
class Memory(BaseModel):
    id: str
    user_id: str
    content: str
    memory_type: str  # fact, preference, conversation
    importance: float
    created_at: datetime
    embeddings: List[float]
```

### Database Schema (PostgreSQL)
```sql
CREATE TABLE memories (
    id UUID PRIMARY KEY,
    user_id VARCHAR(255),
    content TEXT,
    memory_type VARCHAR(50),
    importance FLOAT,
    created_at TIMESTAMP,
    embeddings VECTOR(384)
);

CREATE INDEX idx_user_id ON memories(user_id);
CREATE INDEX idx_importance ON memories(importance DESC);
```

---

## 6. CURRENT USAGE & ADOPTION

### Publicly Known Users
Based on available information:

| Company/Project | Usage |
|----------------|-------|
| **OpenClaw** | Memori integrated as memory plugin for OpenClaw AI agent |
| **Various AI Startups** | Early adopters building AI assistants |
| **Developers** | Individual developers building agentic applications |

### GitHub Statistics
- **Stars**: 11.9k (on Memori repository)
- **Forks**: 951
- **Contributors**: Active open source community

### Integration Ecosystem
- Available via npm: `@aisuite/chub` (for Context Hub)
- SQL-native approach allows custom integrations
- REST API for any language

---

## 7. PRICING MODEL

Memori Cloud offers:
- **Free Tier**: Limited usage for developers
- **Pro Tier**: $X/month for production apps
- **Enterprise**: Custom pricing with dedicated support

---

## 8. LIMITATIONS OF MEMORI

| Limitation | Description |
|-----------|-------------|
| **Individual focus** | Manages memory per user, not organizational |
| **No authority modeling** | Doesn't track who can do what |
| **No commitment tracking** | Doesn't track organizational commitments |
| **Post-facto recall** | Operates after the fact, not pre-action |
| **No multi-agent coordination** | Doesn't help agents coordinate with each other |
| **No org structure** | Doesn't model organizational relationships |

---

# GENIOS vs MEMORI - DETAILED COMPARISON

---

## 1. FUNDAMENTAL DIFFERENCE

| Aspect | Memori | GeniOS |
|--------|--------|--------|
| **Layer** | Memory Layer | Context Graph |
| **Scope** | Individual user/agent | Entire organization |
| **Problem** | Agent amnesia | Agent coordination + compliance |
| **When operates** | Recall past | Verify before action |
| **Data** | Conversations, facts | Relationships, authority, commitments |

---

## 2. ARCHITECTURE COMPARISON

### Memori Architecture
```
Agent → Memory Search → SQL DB → Context Injection → LLM
         ↑                                   
    (After the fact)                        
```

### GeniOS Architecture
```
Agent → Authority Check → Org Graph → Commitment Check → Approve/Deny
         ↑
    (Before action)
```

---

## 3. VALUE PROP COMPARISON

| Value | Memori | GeniOS |
|-------|--------|--------|
| Continuity | ✅ | ✅ |
| Cost reduction | ✅ | ❌ |
| Multi-agent coordination | ❌ | ✅ |
| Authority verification | ❌ | ✅ |
| Compliance enforcement | ❌ | ✅ |
| Commitment tracking | ❌ | ✅ |
| Pre-action checks | ❌ | ✅ |

---

## 4. CUSTOMER COMPARISON

| Customer Type | Memori | GeniOS |
|---------------|--------|--------|
| Individual developers | ✅ | ❌ |
| Startups building AI apps | ✅ | ❌ |
| Enterprises with multiple agents | ⚠️ | ✅ |
| Regulated industries | ❌ | ✅ |
| Compliance-focused orgs | ❌ | ✅ |

---

## 5. FUTURE SCOPE

### Memori's Future Path (Likely)
- Better multi-agent memory
- Enhanced semantic search
- More integrations
- Cost optimization

### GeniOS's Future Path (Your Vision)
- Real-time org state tracking
- Commitment resolution
- Authority graph expansion
- Cross-org coordination (agent-to-agent)
- Physical AI context (robotics)

---

## 6. COEXISTENCE

These products can work **together**:

```
┌─────────────────────────────────────────────────────────────┐
│                      COMPLETE STACK                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              GENIOS (Context Graph)                  │   │
│  │  - Authority verification                            │   │
│  │  - Commitment tracking                                │   │
│  │  - Pre-action verification                           │   │
│  └─────────────────────────────────────────────────────┘   │
│                            ↑                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              ORCHESTRATOR                            │   │
│  │  - Agent coordination                                │   │
│  │  - Task delegation                                   │   │
│  └─────────────────────────────────────────────────────┘   │
│                            ↑                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              MEMORI (Memory Layer)                   │   │
│  │  - Conversation history                             │   │
│  │  - User preferences                                   │   │
│  │  - Facts extraction                                  │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

# DETAILED COMPETITIVE POSITIONING FOR VC CONVERSATIONS

---

## How to Position Against Memori

### The Answer

> "GeniOS operates at a completely different layer than memory products like Memori. Memory layers help agents remember what users said in past conversations - they're solving the 'amnesia problem.' GeniOS solves a fundamentally different problem: when companies deploy multiple AI agents, those agents need to understand the organization they're operating in. They need to know who has authority to approve what, what commitments the company has made, and what the current operational state is.
> 
> An agent can have perfect memory of every conversation it ever had but still make dangerous, organization-breaking mistakes if it doesn't understand organizational context. We're building the layer that prevents those mistakes - not by storing more conversations, but by modeling the organization itself."

### Key Talking Points for VCs

1. **Different Layer**: Memori = memory; GeniOS = organizational context
2. **Different Problem**: Amnesia vs. Coordination/Compliance
3. **Different Buyer**: Developers vs. Enterprise CTO/CISO
4. **Not Either/Or**: Customers likely need both
5. **GeniOS Is Unreplaceable**: Memory can be commoditized; org-specific context compounds over time

---

## Why GeniOS Is Defensible

### The Compounding Data Argument

| Aspect | Memori | GeniOS |
|--------|--------|--------|
| **Data Type** | Generic conversation patterns | Organization-specific context |
| **Switching Cost** | Low - export/import memories | High - months of org behavioral data |
| **Moat** | Feature differentiation | Irreplaceable organizational intelligence |

**Key Insight**: The longer an organization runs GeniOS, the more accurate its organizational graph becomes. Canceling means destroying months of accumulated "organizational truth" that exists nowhere else.

---

# MARKET OPPORTUNITY

---

## TAM Analysis

| Segment | Memori | GeniOS |
|---------|--------|--------|
| **Target** | Developers building AI apps | Enterprises deploying multiple agents |
| **Market Size** | $5B (developer tools) | $50B+ (enterprise AI infrastructure) |
| **Growth** | Fast | Faster (emerging category) |

---

## Competitive Landscape Summary

| Category | Players | GeniOS Position |
|----------|---------|-----------------|
| Memory Layer | mem0, Memori, Zep, Supermemory | Different layer - we sit above them |
| Context Layer | RAG providers | Different - we model org structure |
| Orchestrator | LangGraph, AutoGen | Complementary - we provide context they reference |
| Context Graph | (Emerging players) | Direct competition - but different approach |

---

*Document prepared for GeniOS strategic planning*
*For internal use only*
*Last updated: March 2026*
