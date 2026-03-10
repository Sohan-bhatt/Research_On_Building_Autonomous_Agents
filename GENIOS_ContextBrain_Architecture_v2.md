# GENIOS — Context Brain Architecture
## Organisational Context Graph · Custom Memory Layer · Agent Routing & Orchestration

> **Core Mission:** GeniOS builds an **Organisational Context Graph** for AI Agents — a live, structured model of a company's relationships, commitments, authority structures, and operational state. Agents reference this graph before acting, enabling coordinated decisions, safer automation, and reduced human supervision.
>
> This document defines the full architecture for the Context Brain: the custom memory layer, agent design, routing tables, and orchestration flow.

---

## Table of Contents

1. [Why This Architecture Exists](#1-why-this-architecture-exists)
2. [System Overview](#2-system-overview)
3. [The Three-Layer Model (Where GeniOS Lives)](#3-the-three-layer-model-where-genios-lives)
4. [Custom Memory Layer Architecture](#4-custom-memory-layer-architecture)
5. [Organisational Context Graph (The Core)](#5-organisational-context-graph-the-core)
6. [Memory Lifecycle & Operations](#6-memory-lifecycle--operations)
7. [Agent Architecture](#7-agent-architecture)
8. [Routing Table Design](#8-routing-table-design)
9. [Orchestration Diagram](#9-orchestration-diagram)
10. [Technology Stack](#10-technology-stack)
11. [Implementation Roadmap](#11-implementation-roadmap)

---

## 1. Why This Architecture Exists

### The Three Problems We Solve

| Problem | What It Means | Architectural Answer |
|---|---|---|
| **Agents Start Blind** | Every agent begins a task with zero understanding of the org — no relationships, no authority, no commitments, no live state | Organisational Context Graph injected into every agent before it acts |
| **Agents Cannot Coordinate** | Multiple agents deployed across a company make conflicting decisions because they share no common reality | Shared graph layer — all agents read from the same org model simultaneously |
| **Humans Must Supervise Everything** | Companies keep humans in the loop because agents cannot safely act without knowing rules, past commitments, and decision history | Authority + commitment modeling inside the graph — agents self-check before acting |

### What Separates GeniOS from Competitors

```
Memory Layer (Mem0, Zep, Supermemory)
  → Answers: "What happened before?"
  → Scope: Single user or single agent

Context Layer (RAG, Hyperspell, UseCortex)
  → Answers: "What information is relevant?"
  → Scope: Documents and knowledge retrieval

Context Graph (GeniOS)
  → Answers: "What does the organisation actually look like RIGHT NOW,
              and what is this agent permitted to do about it?"
  → Scope: The entire organisation — people, tools, workflows, commitments,
           authority structures, live state — shared across ALL agents
```

> **The key insight:** We are not building a smarter memory wrapper. We are building the organisational intelligence layer that no orchestrator, RAG system, or memory API can provide.

---

## 2. System Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│               ORGANISATIONAL TOOLS (Source of Truth)                     │
│    Slack · Email · CRM · Calendar · Project Tools · HR Systems           │
└───────────────────────────────────┬──────────────────────────────────────┘
                                    │ continuous sync
                                    ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                     GENIOS CONTEXT BRAIN                                 │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │              ORGANISATIONAL CONTEXT GRAPH                        │    │
│  │   Entities · Relationships · Authority · Commitments · State     │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  ┌──────────────────────┐    ┌────────────────────────────────────┐     │
│  │    MEMORY LAYER      │    │        ORCHESTRATION LAYER         │     │
│  │  Working · Episodic  │    │   Router · Planner · Dispatcher    │     │
│  │  Semantic · Procedural│   │                                    │     │
│  └──────────────────────┘    └────────────────────────────────────┘     │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                        AGENT NETWORK                             │   │
│  │  Graph Agent · Sync Agent · Research · Synthesis · Authority     │   │
│  │  Coordinator · Critic · Report                                   │   │
│  └──────────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌──────────────────────────────────────────────────────────────────────────┐
│              AGENT CONSUMERS (Internal or External)                      │
│    Sales Agent · Finance Agent · Executive Assistant · Ops Agent         │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 3. The Three-Layer Model (Where GeniOS Lives)

GeniOS is NOT a memory layer. It is NOT a RAG system. It sits ABOVE both.

```
┌─────────────────────────────────────────────────────────────────────┐
│  LAYER 3 — ORGANISATIONAL CONTEXT GRAPH (GeniOS)                   │
│                                                                     │
│  Models the org. Relationships, authority, commitments, live state. │
│  All agents share one version of organisational truth.              │
│  Continuously updated as the org changes.                           │
├─────────────────────────────────────────────────────────────────────┤
│  LAYER 2 — CONTEXT / RETRIEVAL LAYER                                │
│                                                                     │
│  Fetches relevant documents and knowledge chunks.                   │
│  Answers "what information exists?"                                 │
│  Built into most LLM stacks. Becoming commoditised.                 │
├─────────────────────────────────────────────────────────────────────┤
│  LAYER 1 — MEMORY LAYER                                             │
│                                                                     │
│  Stores past interactions, preferences, conversation history.       │
│  Answers "what happened before?"                                    │
│  Easy to build. Easy to replicate. Not a moat.                      │
└─────────────────────────────────────────────────────────────────────┘
```

**GeniOS builds Layer 3 — and subsumes Layer 1 and Layer 2 as internal components of the Context Brain, not as external dependencies.**

---

## 4. Custom Memory Layer Architecture

The Memory Layer is a supporting system INSIDE the Context Brain. It is not the product — it is infrastructure that feeds the Context Graph. We build it ourselves (no Mem0, Zep, or Letta dependency).

### 4.1 Four Memory Tiers

```
┌─────────────────────────────────────────────────────────────────────┐
│                        MEMORY LAYER                                 │
│                                                                     │
│  ┌──────────────────────┐  ┌──────────────────────────────────────┐ │
│  │   WORKING MEMORY     │  │         EPISODIC MEMORY              │ │
│  │                      │  │                                      │ │
│  │  Active session      │  │  Past agent sessions, decisions,     │ │
│  │  context window.     │  │  outcomes. What happened and what    │ │
│  │  In-flight task      │  │  was decided across the org.         │ │
│  │  state. Agent chain  │  │  Searchable by semantic similarity.  │ │
│  │  progress.           │  │                                      │ │
│  │  Storage: Redis      │  │  Storage: PostgreSQL + Vector DB     │ │
│  └──────────────────────┘  └──────────────────────────────────────┘ │
│                                                                     │
│  ┌──────────────────────┐  ┌──────────────────────────────────────┐ │
│  │   SEMANTIC MEMORY    │  │       PROCEDURAL MEMORY              │ │
│  │                      │  │                                      │ │
│  │  Org entities: who   │  │  Agent playbooks. Repeatable         │ │
│  │  owns what, what     │  │  workflows. Learned patterns from    │ │
│  │  commitments exist,  │  │  past successful task chains.        │ │
│  │  what rules apply.   │  │  SOPs for agent coordination.        │ │
│  │  User preferences.   │  │                                      │ │
│  │                      │  │  Storage: PostgreSQL + JSON config   │ │
│  │  Storage: PostgreSQL │  │                                      │ │
│  │  + Vector DB + Graph │  │                                      │ │
│  └──────────────────────┘  └──────────────────────────────────────┘ │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              MEMORY CONTROLLER (Core Engine)                │   │
│  │    Write → Extract → Embed → Store → Retrieve → Inject      │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### 4.2 Memory Tier Details

#### Working Memory (In-Session)
**Storage:** Redis (TTL: session duration)
```json
{
  "session_id": "sess_abc123",
  "org_id": "org_xyz",
  "agent_chain": ["router_agent", "authority_agent", "sales_agent"],
  "active_task": "review_vendor_contract",
  "graph_snapshot": { "vendor": "Acme", "budget_status": "frozen", "approver": "CFO" },
  "messages": [...],
  "ttl": 3600
}
```

#### Episodic Memory (Interaction History)
**Storage:** PostgreSQL + Vector DB
```json
{
  "episode_id": "ep_001",
  "org_id": "org_xyz",
  "timestamp": "2025-03-10T10:00:00Z",
  "summary": "Finance agent attempted vendor contract approval. CFO authority required. Routed for human approval.",
  "agents_involved": ["router_agent", "authority_agent", "finance_agent"],
  "org_state_snapshot": { "budget_freeze": true, "pending_approvals": 3 },
  "outcome": "escalated_to_human",
  "embedding": [...]
}
```

#### Semantic Memory (Org Facts + Entities)
**Storage:** PostgreSQL (structured) + Neo4j or pgvector (graph relations) + Qdrant (unstructured)

| Sub-layer | Content | Storage |
|---|---|---|
| Entity Store | People, teams, vendors, projects, tools | PostgreSQL + Graph DB |
| Commitment Store | Active contracts, promises, deadlines | PostgreSQL |
| Authority Map | Who can approve what, org hierarchy, delegation rules | PostgreSQL + Graph |
| Operational State | Budget status, active freezes, workflow states | PostgreSQL (live-updated) |
| Knowledge Chunks | Policies, SOPs, unstructured org docs | Vector DB (Qdrant) |

#### Procedural Memory (Agent Playbooks)
**Storage:** PostgreSQL + versioned JSON
```json
{
  "procedure_id": "proc_vendor_approval",
  "name": "Vendor Contract Approval Workflow",
  "trigger_conditions": ["vendor", "contract", "approve", "sign"],
  "authority_check_required": true,
  "steps": [
    { "step": 1, "agent": "authority_agent", "action": "check_approval_rights" },
    { "step": 2, "agent": "graph_agent", "action": "check_budget_state" },
    { "step": 3, "agent": "coordinator_agent", "action": "route_or_escalate" }
  ],
  "success_count": 87,
  "avg_latency_ms": 1800
}
```

---

## 5. Organisational Context Graph (The Core)

This is the product. The graph models what the organisation actually looks like right now.

### 5.1 Graph Schema

```
NODES (Entity Types)
─────────────────────────────────────────────
  Person        → Employee, role, seniority, team
  Team          → Department, function, hierarchy
  Vendor        → External company, contract status, relationship stage
  Project       → Active/archived, owner, dependencies
  Commitment    → Signed agreements, verbal promises, deadlines
  Budget        → Allocated funds, spend, freeze status
  Tool          → Connected software systems (Slack, CRM, email)
  Decision      → Logged decisions, who made them, when, why
  Rule          → Org policy, approval thresholds, constraints

EDGES (Relationship Types)
─────────────────────────────────────────────
  REPORTS_TO       Person → Person
  OWNS             Person/Team → Project/Budget/Tool
  HAS_AUTHORITY    Person → Decision (with threshold)
  COMMITTED_TO     Person/Team → Vendor/Commitment
  DEPENDS_ON       Project → Project/Person/Tool
  GOVERNS          Rule → Person/Team/Budget/Action
  CONNECTED_VIA    Tool → Tool (data flow)
  MADE_DECISION    Person → Decision
  IN_STATE         Budget/Project → OperationalState
```

### 5.2 Graph Query Examples (What Agents Ask)

Before any action, agents query the graph:

```
QUERY: "Can agent X approve this vendor contract?"
GRAPH ANSWER:
  → Vendor: Acme Corp (relationship_stage: negotiation)
  → Contract value: $120,000
  → Authority threshold for contracts > $100k: CFO approval required
  → Current CFO: Jane Doe (available: true, delegation: none active)
  → Budget state: frozen (since 2025-03-01)
  → VERDICT: Cannot approve. Escalate to CFO. Budget freeze blocks disbursement.

QUERY: "Is there a conflicting commitment with Vendor B?"
GRAPH ANSWER:
  → Vendor B has an active exclusivity clause (expires: 2025-06-30)
  → Proposed action (sign with Vendor A) violates exclusivity clause
  → VERDICT: Block action. Surface conflict to human.
```

### 5.3 Graph Sync Architecture

The graph must stay live. It syncs continuously from org tools:

```
┌─────────────────────────────────────────────────────────────────┐
│                    SYNC PIPELINE                                │
│                                                                 │
│  Slack ──────────────────────────────┐                         │
│  Email (Gmail / Outlook) ────────────┤                         │
│  CRM (Salesforce / HubSpot) ─────────┤──► SYNC AGENT          │
│  Calendar (Google / Outlook) ────────┤       │                 │
│  Project Tools (Jira / Notion) ──────┤       │ Extract         │
│  HR Systems ─────────────────────────┘       │ entities,       │
│                                              │ relationships,  │
│                                              │ events,         │
│                                              │ state changes   │
│                                              ▼                 │
│                                    ┌──────────────────┐        │
│                                    │  GRAPH UPDATER   │        │
│                                    │  (write to Neo4j │        │
│                                    │   + PostgreSQL)  │        │
│                                    └──────────────────┘        │
└─────────────────────────────────────────────────────────────────┘
```

**Sync Modes:**
- **Real-time webhooks** — for high-signal events (contract signed, budget frozen, person leaves)
- **Scheduled polling** — for slower-changing data (org structure, tool integrations)
- **On-demand refresh** — triggered by agent before acting on sensitive data

---

## 6. Memory Lifecycle & Operations

### 6.1 Write Path

```
Agent Output / Tool Event / Org System Event
              │
              ▼
    ┌──────────────────┐
    │ MEMORY EXTRACTOR │  ← LLM-powered: extract entities, relationships,
    └────────┬─────────┘    commitments, decisions from unstructured text
             │
    ┌────────┴────────────────────────────┐
    │                                     │
    ▼                                     ▼
┌──────────────┐                 ┌──────────────────┐
│   Embedder   │                 │  Entity + Relation│
│ (text → vec) │                 │  Extractor (NER + │
└──────┬───────┘                 │  graph linking)   │
       │                         └────────┬──────────┘
       ▼                                  ▼
┌──────────────────────────────────────────────────────┐
│              MEMORY ROUTER (Write)                   │
│  Decides: Working / Episodic / Semantic / Graph      │
└──────────────────────────────────────────────────────┘
       │           │            │              │
       ▼           ▼            ▼              ▼
    Redis      PostgreSQL    Vector DB       Neo4j
  (working)   (episodic /   (knowledge     (org graph)
               semantic)     chunks)
```

### 6.2 Read Path (Context Injection)

```
Incoming Agent Request
        │
        ▼
┌────────────────────────┐
│    QUERY ANALYZER      │  ← Intent + entity extraction from request
└──────────┬─────────────┘
           │
    ┌──────┴──────────────────────────────────────┐
    │          │              │                   │
    ▼          ▼              ▼                   ▼
 Working    Episodic       Semantic           Context
 Memory     Search         Search             Graph
 (Redis)  (vector sim    (fact lookup)      Query (Neo4j)
           over past       (PostgreSQL        Relationships,
           sessions)       + Qdrant)          authority,
                                             live state
    │          │              │                   │
    └──────────┴──────────────┴───────────────────┘
                              │
                    ┌─────────▼──────────┐
                    │   MEMORY MERGER    │
                    │  Re-rank, dedupe,  │
                    │  compress, format  │
                    └─────────┬──────────┘
                              │
                    CONTEXT PACKAGE injected into Agent prompt
```

### 6.3 Session End — Memory Consolidation

```
Session ends
     │
     ├──► Summarise session (LLM)
     ├──► Extract new entities + relationships → update Graph
     ├──► Extract new commitments → write to Commitment Store
     ├──► Write episode → Episodic Memory
     ├──► Detect authority decisions made → log to Decision Store
     ├──► Update Procedural Memory if new pattern found
     └──► Flush Working Memory (Redis)
```

---

## 7. Agent Architecture

All agents are **stateless**. They do not hold memory. Memory lives in the Memory Layer and the Context Graph. The Orchestrator injects context into every agent before it acts.

### 7.1 Agent Registry

| Agent ID | Name | Responsibility | Key Inputs | Key Outputs |
|---|---|---|---|---|
| `router_agent` | Router | Classify intent, select agent chain, check routing table | Raw input + working memory | Agent chain + DAG |
| `graph_agent` | Graph Query Agent | Query the org context graph for relationships, authority, state | Graph query params | Org context package |
| `sync_agent` | Graph Sync Agent | Pull data from org tools, update the graph continuously | Tool webhooks / API events | Graph write operations |
| `authority_agent` | Authority Checker | Determine if an action is permitted given current org state | Action + actor + graph context | Permit / Block / Escalate |
| `coordinator_agent` | Coordinator | Route multi-agent tasks, manage conflicts between agents | Task + graph state | Resolved task plan |
| `research_agent` | Researcher | Web search, internal doc retrieval, RAG over org knowledge | Query + semantic memory | Structured findings |
| `synthesis_agent` | Synthesizer | Summarise, compare, analyse across multiple sources | Findings + context | Analysis output |
| `memory_agent` | Memory Manager | Explicit memory read/write operations | Memory commands | Memory read/write results |
| `critic_agent` | Critic | Quality gate — check output for org-context violations | Agent output + org rules | Pass / Fail + corrections |
| `report_agent` | Reporter | Format final output for human or downstream agent | Synthesis output | Formatted response |
| `planner_agent` | Planner | Decompose complex tasks into DAG of agent steps | Complex task + procedural memory | Execution DAG |

### 7.2 Standard Agent Interface

```python
class BaseAgent:
    agent_id: str
    capabilities: List[str]
    required_context_keys: List[str]  # what it pulls from memory/graph

    def execute(
        self,
        task: Task,
        context: ContextPackage,       # injected by orchestrator
        graph_context: OrgGraphSlice,  # injected org graph snapshot
        tools: List[Tool]
    ) -> AgentOutput:
        ...

    def on_complete(self, output: AgentOutput) -> MemoryWriteRequest:
        # What to write back to memory + graph after task
        ...

    def authority_check(self, action: Action, graph: OrgGraphSlice) -> AuthorityVerdict:
        # Every agent checks authority before executing sensitive actions
        ...
```

---

## 8. Routing Table Design

### 8.1 Intent Classification

```
User / System Input
        │
        ▼
┌────────────────────────────────────┐
│       INTENT CLASSIFIER            │
│  (Fast LLM call or fine-tuned      │
│   small model — latency critical)  │
│                                    │
│  Output:                           │
│  { intent, confidence,             │
│    entities, org_sensitivity,      │
│    authority_required,             │
│    complexity }                    │
└────────────────┬───────────────────┘
                 │
                 ▼
           ROUTING TABLE
```

### 8.2 Master Routing Table

| Intent | Confidence Threshold | Authority Check | Primary Agent | Supporting Agents | Memory Reads | Graph Query | Escalate If |
|---|---|---|---|---|---|---|---|
| `simple_qa` | > 0.85 | No | `synthesis_agent` | — | Working, Semantic | No | complexity > 3 |
| `org_context_query` | > 0.80 | No | `graph_agent` | `synthesis_agent` | Episodic, Semantic | Yes (full) | — |
| `authority_sensitive_action` | > 0.80 | **Yes** | `authority_agent` | `graph_agent`, `coordinator_agent` | All layers | Yes (authority + state) | always |
| `multi_agent_coordination` | > 0.75 | **Yes** | `coordinator_agent` | All relevant agents | All layers | Yes (full org slice) | always |
| `research_task` | > 0.80 | No | `research_agent` | `synthesis_agent`, `report_agent` | Episodic, Semantic | Optional | multi-step detected |
| `commitment_check` | > 0.85 | No | `graph_agent` | `authority_agent` | Semantic (commitments) | Yes (commitments + vendors) | conflict found |
| `memory_recall` | > 0.90 | No | `memory_agent` | — | All layers | No | — |
| `memory_save` | > 0.90 | No | `memory_agent` | `graph_agent` (entity link) | Working | Yes (entity resolution) | — |
| `complex_task` | > 0.70 | **Yes** | `planner_agent` | All agents as needed | All layers | Yes (full) | always |
| `report_generation` | > 0.80 | No | `planner_agent` | `research_agent`, `synthesis_agent`, `report_agent` | Episodic, Semantic, Procedural | Optional | — |
| `graph_sync_event` | > 0.90 | No | `sync_agent` | `graph_agent` | Working | Yes (write) | sync conflict |
| `clarification_needed` | < 0.60 | No | `router_agent` | — | Working | No | — |
| `unknown` | any | **Yes** | `planner_agent` | `critic_agent` | All layers | Yes | always |

### 8.3 Routing Decision Flow

```
┌────────────────────────────────────────────────────────────────────┐
│                         ROUTER AGENT                               │
│                                                                    │
│  1. Load Working Memory (current session)                          │
│  2. Run Intent Classifier → { intent, confidence, sensitivity }    │
│  3. Evaluate confidence:                                           │
│     ├── HIGH  → Direct route via routing table                     │
│     ├── MEDIUM → Route + mandatory critic_agent quality gate       │
│     └── LOW   → Ask clarification OR escalate to planner_agent    │
│  4. Check org_sensitivity flag:                                    │
│     └── If TRUE → always prepend authority_agent to chain         │
│  5. Query graph for relevant org context slice                     │
│  6. Build ContextPackage (memory + graph) → inject into agents     │
│  7. Dispatch agent chain / DAG                                     │
│  8. Collect outputs → write to memory + update graph              │
└────────────────────────────────────────────────────────────────────┘
```

### 8.4 Authority Gate (GeniOS-Specific)

This is unique to GeniOS. Before any action that touches the org (contracts, approvals, commitments, decisions), the **Authority Agent** runs a graph check:

```
Action Proposed by Agent
          │
          ▼
  ┌────────────────────┐
  │  AUTHORITY AGENT   │
  │                    │
  │  Queries graph:    │
  │  ─ Who owns this?  │
  │  ─ Approval limit? │
  │  ─ Active freezes? │
  │  ─ Conflicts?      │
  │  ─ Delegation?     │
  └────────┬───────────┘
           │
    ┌──────┴──────────────────┐
    │                         │
    ▼                         ▼                    ▼
PERMITTED               BLOCKED                ESCALATE
(agent proceeds)    (action stopped,       (route to human
                     reason logged)         or senior agent)
```

---

## 9. Orchestration Diagram

```
══════════════════════════════════════════════════════════════════════════
                     GENIOS CONTEXT BRAIN
                  FULL ORCHESTRATION FLOW
══════════════════════════════════════════════════════════════════════════

ORGANISATIONAL TOOLS (Slack, Email, CRM, Calendar, Jira, HR)
      │
      │ webhooks / scheduled sync
      ▼
╔═════════════════════╗
║    SYNC AGENT       ║  ← Continuously updates the Org Graph
╚══════════╤══════════╝    from all connected tools
           │ graph writes
           ▼
╔═════════════════════════════════════════════════════════════╗
║              ORGANISATIONAL CONTEXT GRAPH                   ║
║                                                             ║
║  Entities · Relationships · Authority · Commitments · State ║
║  Neo4j + PostgreSQL (live, continuously updated)            ║
╚══════════════════════════════════════════════════════════════╝
           ▲                          │
           │ graph writes             │ graph reads (context injection)
           │                          ▼
═══════════════════════════════════════════════════════════════
USER / API / DOWNSTREAM AGENT INPUT
      │
      ▼
╔═════════════════════╗
║   ORCHESTRATOR      ║  ← Stateless. Manages full request lifecycle.
╚══════════╤══════════╝
           │ 1. load session context
           ▼
╔═════════════════════╗
║    MEMORY LAYER     ║◄────────────────────────────────────────────┐
║  ───────────────    ║                                             │
║  Working (Redis)    ║                                             │
║  Episodic (PG+Vec)  ║                                             │
║  Semantic (PG+Vec)  ║                                             │
║  Procedural (PG)    ║                                             │
╚══════════╤══════════╝                                             │
           │ 2. ContextPackage returned                             │
           ▼                                                        │
╔═════════════════════╗                                             │
║    ROUTER AGENT     ║  ← Intent classify + routing table         │
║  ───────────────    ║    lookup → returns AgentChain + DAG       │
║  Intent Classifier  ║                                             │
║  Routing Table      ║                                             │
║  Org Sensitivity    ║                                             │
║  Check              ║                                             │
╚══════════╤══════════╝                                             │
           │ 3a. If org-sensitive → prepend authority gate          │
           ▼                                                        │
╔═════════════════════╗                                             │
║  AUTHORITY AGENT    ║  ← Queries org graph: who can do what      │
║  ───────────────    ║    right now given live org state          │
║  Graph Query        ║    → PERMIT / BLOCK / ESCALATE             │
║  Authority Map      ║                                             │
║  Commitment Check   ║                                             │
╚══════════╤══════════╝                                             │
           │ 3b. If PERMITTED → dispatch agent DAG                  │
           ▼                                                        │
╔═══════════════════════════════════════════════════════════════╗   │
║                     AGENT NETWORK                             ║   │
║                                                               ║   │
║  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐             ║   │
║  │  graph      │ │  research   │ │  planner    │             ║   │
║  │  _agent    │ │  _agent    │ │  _agent    │             ║   │
║  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘             ║   │
║         │               │               │                     ║   │
║  ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐             ║   │
║  │coordinator  │ │  synthesis  │ │   memory    │             ║   │
║  │  _agent    │ │  _agent    │ │  _agent    │             ║   │
║  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘             ║   │
║         │               │               │                     ║   │
║         └───────────────┴───────────────┘                     ║   │
║                         │                                     ║   │
║               ┌──────────▼──────────┐                         ║   │
║               │    critic_agent    │ ← quality + org-rule    ║   │
║               └──────────┬──────────┘   violation check      ║   │
║                          │                                    ║   │
║               ┌──────────▼──────────┐                         ║   │
║               │    report_agent    │                         ║   │
║               └──────────┬──────────┘                         ║   │
╚══════════════════════════╪════════════════════════════════════╝   │
                           │                                        │
                           │ 4. Write outputs → memory + graph ────►│
                           │
                           │ 5. Return final output
                           ▼
                    USER / DOWNSTREAM AGENT RESPONSE

══════════════════════════════════════════════════════════════════════════
```

### 9.1 Multi-Agent DAG Example

**Task:** "Review and approve the Acme vendor contract"

```
router_agent (classify: authority_sensitive_action)
     │
     ▼
authority_agent ──── queries graph ────► budget: frozen, CFO approval needed
     │
     ├──[parallel]──► graph_agent (pull Acme relationship, commitment history)
     │
     ├──[parallel]──► research_agent (pull contract doc from knowledge store)
     │
     └──[after both]──► synthesis_agent (analyse contract vs org commitments)
                              │
                              ▼
                        coordinator_agent (conflict check across agents)
                              │
                              ▼
                         critic_agent (org-rule violation check)
                              │
                   ┌──────────┴──────────┐
                   ▼                     ▼
             PERMIT path           ESCALATE path
          report_agent         route to CFO (human)
          (output summary)     with full context package
```

---

## 10. Technology Stack

| Component | Technology | Why |
|---|---|---|
| **Org Context Graph** | Neo4j (graph DB) | Native graph traversal for relationship + authority queries |
| **Structured Org Data** | PostgreSQL | Commitments, authority maps, entity tables, operational state |
| **Working Memory** | Redis | Ultra-fast, TTL-native, session scoped |
| **Episodic + Semantic (unstructured)** | Qdrant (self-hosted) | Fast ANN vector search, open-source, no vendor lock |
| **Embeddings** | `text-embedding-3-small` or `nomic-embed` (local) | Cost efficiency; swappable |
| **LLM (agents)** | Claude Sonnet / GPT-4o (via API) | Swappable per agent; no lock-in |
| **Intent Classifier** | Fine-tuned Llama 3 8B or prompted Sonnet | Speed-critical first hop |
| **Memory Extractor** | LLM + Pydantic structured output | Entity + relationship extraction from free text |
| **Orchestrator** | Python (FastAPI, async) | Lightweight, async DAG execution |
| **Agent Runtime** | Python (custom, no LangChain) | Full control; no framework lock-in |
| **Sync Pipeline** | Webhooks + Redis Streams | Real-time tool event ingestion into graph |
| **Graph Sync (scheduled)** | Celery + Redis | Periodic polling for slower-changing org data |
| **Message Queue (multi-agent)** | Redis Streams or Kafka | Async DAG coordination between agents |

---

## 11. Implementation Roadmap

### Phase 1 — Memory Foundation (Weeks 1–3)
- [ ] Stand up PostgreSQL + Redis + Qdrant
- [ ] Build Memory Controller (read/write/embed APIs)
- [ ] Implement Working Memory (session store in Redis)
- [ ] Implement Semantic Memory (entity store + user profile)
- [ ] Build Embedder service
- [ ] Build Memory Extractor (LLM-powered, Pydantic structured output)

### Phase 2 — Org Graph Foundation (Weeks 4–6)
- [ ] Stand up Neo4j; define graph schema (entities, relationships, authority)
- [ ] Build Graph Query API (authority checks, state queries, relationship traversal)
- [ ] Build Sync Agent (connect first org tool — start with email or Slack)
- [ ] Implement Graph Updater (writes extracted entities/events into Neo4j)
- [ ] Build Authority Agent (graph-backed authority + commitment checks)

### Phase 3 — Router + Agent Network (Weeks 7–9)
- [ ] Define BaseAgent interface (context injection, authority_check, on_complete)
- [ ] Implement Router Agent + Intent Classifier + Routing Table
- [ ] Implement Research Agent + Synthesis Agent
- [ ] Implement Graph Agent (query interface for agents)
- [ ] Wire full flow: Input → Router → Authority Gate → Agent → Memory+Graph Write → Output

### Phase 4 — Orchestration + Multi-Agent (Weeks 10–12)
- [ ] Build Orchestrator (DAG execution engine, parallel + sequential)
- [ ] Implement Planner Agent (task decomposition into DAGs)
- [ ] Implement Coordinator Agent (conflict resolution across agents)
- [ ] Implement Critic Agent (org-rule violation quality gate)
- [ ] Implement Episodic Memory + Memory Consolidator (session-end job)
- [ ] Add Procedural Memory (save + reuse successful agent workflows)

### Phase 5 — Graph Compounding + Evaluation (Weeks 13–16)
- [ ] Connect remaining org tools (CRM, Calendar, Jira, HR)
- [ ] Build Graph drift detection (alert when org state becomes stale)
- [ ] Evaluation framework: authority accuracy, retrieval precision, agent success rate, graph freshness
- [ ] Dynamic Routing Table updates (confidence-based learning)
- [ ] Dashboard: graph health, agent performance, memory hit rates

---

## Key Design Principles

**1. Graph First, Memory Second**
The Org Context Graph is the product. The Memory Layer is infrastructure that feeds it. Every architectural decision should ask: "Does this make the graph more accurate and useful?"

**2. Authority Before Action**
No org-sensitive action executes without an authority check against the live graph. This is the safety layer competitors cannot replicate without the graph.

**3. Shared Truth Across Agents**
All agents read from the same graph at the same moment. No agent builds its own partial view of the org. Conflicts are impossible if the graph is the single source of truth.

**4. No Framework Lock-in**
No LangChain, no Mem0, no Zep, no Letta. Every component is owned. The moat is the data and the graph, not the framework.

**5. The Graph Compounds**
The longer GeniOS runs inside an org, the more accurate the graph becomes. Twelve months of real org behavioral data cannot be copied by a competitor. This is the real moat.

---

*Document version: 2.0 | Aligned with GENIOS Competitive Landscape | March 2026*
