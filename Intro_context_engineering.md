# 🧠 Context Engineering & Agent Memory 

> **TL;DR:** Context engineering is the art + science of managing *what* an AI agent sees at each step. Better context = better performance, lower cost, fewer failures.

---

## 📌 Table of Contents

1. [What is Context Engineering?](#1-what-is-context-engineering)
2. [Why It Matters for AI Agents](#2-why-it-matters-for-ai-agents)
3. [Core Strategy Overview](#3-core-strategy-overview)
4. [Agent Memory Types](#4-agent-memory-types)
5. [Memory Failure Modes](#5-memory-failure-modes)
6. [Toolkit of Techniques](#6-toolkit-of-techniques)
7. [Short-Term Techniques Deep Dive](#7-short-term-techniques-deep-dive)
8. [Long-Term / Cross-Session Memory](#8-long-term--cross-session-memory)
9. [Prompt & Tool Hygiene](#9-prompt--tool-hygiene)
10. [Agent Context Profiles](#10-agent-context-profiles)
11. [Memory Design Best Practices](#11-memory-design-best-practices)
12. [Evaluating Memory Performance](#12-evaluating-memory-performance)
13. [Scaling Agent Memory Systems](#13-scaling-agent-memory-systems)
14. [Memory Pruning & Temporal Decay](#14-memory-pruning--temporal-decay)
15. [Key Takeaways & Resources](#15-key-takeaways--resources)

---

## 1. What is Context Engineering?

| Dimension | Description |
|-----------|-------------|
| **Art** | Judgment — knowing *what* information is most relevant at each reasoning step |
| **Science** | Systematic, repeatable patterns with measurable impact |

> 💡 **Core insight:** LLM performance depends heavily on *context quality*, not just the model's inherent capability.

### Disciplines Ecosystem

```mermaid
mindmap
  root((Context Engineering))
    Prompt Engineering
      Clarity
      Structure
      Few-shot examples
    Structured Output
      Predictable formats
      JSON / XML schemas
    RAG
      External knowledge
      Vector retrieval
    State & History Management
      Session tracking
      Turn management
    Memory
      Persistent storage
      Cross-session recall
      Databases / files
```

---

## 2. Why It Matters for AI Agents

Without proper context management, long-running or tool-heavy agents can:

- 🔺 **Bloat tokens** → higher costs + slower processing
- 🌀 **Degrade quality** → confusion, "poisoning noise", context overflow (bursting)

---

## 3. Core Strategy Overview

```mermaid
flowchart TD
    Problem["⚠️ Growing / Polluted Context"] --> S1 & S2 & S3

    S1["🔵 Reshape & Fit\nOptimize to fit within context window"]
    S2["🟢 Isolate & Route\nSend right context to right agent"]
    S3["🟠 Extract & Retrieve\nStore high-quality memories for later"]

    S1 --> T1["Trimming\nCompaction\nSummarization"]
    S2 --> T2["Sub-agent offloading\nSelective handoff"]
    S3 --> T3["Memory extraction\nState management\nMemory retrieval"]
```

---

## 4. Agent Memory Types

```mermaid
flowchart LR
    subgraph ST["🕐 Short-Term Memory (In-Session)"]
        direction TB
        A["Optimize context window\nduring active interaction"]
    end

    subgraph LT["🗂️ Long-Term Memory (Cross-Session)"]
        direction TB
        B["Build continuity\nacross multiple sessions"]
    end

    ST -->|"Session ends → summarize / extract"| LT
    LT -->|"Session starts → inject memories"| ST
```

### The Core Bottleneck

```mermaid
flowchart LR
    Budget["🪣 Fixed Token Budget"]
    Budget --> P["System Prompt"]
    Budget --> H["Conversation History"]
    Budget --> T["Tool Definitions"]
    Budget --> TR["Tool Results"]
    Budget --> M["Injected Memories"]
    Budget --> O["Agent Output"]

    style Budget fill:#ff6b6b,color:#fff
```

> Every component **competes** for the same fixed token budget. This is the fundamental challenge.

---

## 5. Memory Failure Modes

```mermaid
flowchart TD
    F["⚡ Agent Memory Failure Modes"]

    F --> CB["1️⃣ Context Burst\nSudden token spike\ne.g., large tool payload injected"]
    F --> CC["2️⃣ Context Conflict\nContradictory instructions\ne.g., 'no refunds' vs 'VIP eligible'"]
    F --> CP["3️⃣ Context Poisoning\nHallucinated/inaccurate info\npropagates across turns"]
    F --> CN["4️⃣ Context Noise\nRedundant/overlapping content\ne.g., too many similar tool defs"]

    style CB fill:#ffd93d
    style CC fill:#ff6b6b,color:#fff
    style CP fill:#c0392b,color:#fff
    style CN fill:#f39c12,color:#fff
```

### Failure Mode Quick Reference

| Failure | Trigger | Symptom | Fix |
|---------|---------|---------|-----|
| **Burst** | Large tool result dumped | Token spike in one turn | Trim tool outputs; compaction |
| **Conflict** | Contradictory instructions | Wrong action taken | Remove conflicting prompts; clear precedence rules |
| **Poisoning** | Hallucination in summary | Error propagates downstream | Strict summarization prompts; fact-check guards |
| **Noise** | Too many similar tools | Model picks wrong tool | Minimize + differentiate tool set |

---

## 6. Toolkit of Techniques

```mermaid
flowchart TD
    CE["Context Engineering Toolkit"]

    CE --> Short["🕐 Short-Term (In-Session)"]
    CE --> Long["🗂️ Long-Term (Cross-Session)"]

    Short --> RF["Reshape & Fit"]
    Short --> IR["Isolate & Route"]

    RF --> Trim["✂️ Context Trimming\nDrop older turns, keep last N"]
    RF --> Comp["📦 Compaction\nDrop old tool results, keep structure"]
    RF --> Sum["📝 Summarization\nCompress older turns into golden summary"]

    IR --> Sub["🤖 Sub-agent Offloading\nDelegate context + tools to specialists"]
    IR --> Hand["🔀 Selective Handoff\nPass only relevant context between agents"]

    Long --> Ext["🔍 Memory Extraction\nPull key facts into structured store"]
    Long --> State["⚙️ State Management\nTrack goal/status across sessions"]
    Long --> Ret["📡 Memory Retrieval\nVector search → inject relevant memories"]
```

---

## 7. Short-Term Techniques Deep Dive

### Static vs. Dynamic Tokens

| Type | Examples | Control Level |
|------|---------|--------------|
| **Static** | System prompt, tool definitions, examples | Low — fixed per session |
| **Dynamic** | Tool results, memories, conversation history | High — prime targets for optimization |

> 🎯 Focus optimization efforts on **dynamic tokens** — they scale with session length.

---

### Context Trimming

```mermaid
sequenceDiagram
    participant U as User
    participant A as Agent
    participant M as Memory Manager

    U->>A: Turn 1: Question
    U->>A: Turn 2: Follow-up
    U->>A: Turn 3: New topic
    Note over M: Trigger: N turns reached
    M->>A: DROP turns 1-2 (older turns removed)
    U->>A: Turn 4: Continue
    Note over A: Fresh context from Turn 3 onward ✅
```

**When to use trimming:**
- Tool-heavy agents with independent tasks
- Short workflows where older data won't be needed
- Speed is priority (zero added latency)

**⚠️ Rule:** Never trim mid-turn — always respect full turn blocks (user msg + all agent responses until next user msg).

---

### Context Compaction

Similar to trimming, but **only removes tool call results** (not the structural placeholders).

```
Before compaction:           After compaction:
[Turn 1: User msg]           [Turn 1: User msg]
[Turn 1: Tool call]          [Turn 1: Tool call placeholder ✓]
[Turn 1: Tool result 📦]  →  [Turn 1: Tool result REMOVED ✗]
[Turn 2: Agent response]     [Turn 2: Agent response]
```

**When to use compaction:**
- Tool-heavy workflows where result data is no longer needed
- Keep conversation structure intact
- Reduce noise without losing conversational flow

---

### Context Summarization

```mermaid
flowchart LR
    Old["📜 Turns 1–N\n(older messages)"]
    Recent["🔵 Turns N-3 to N\n(kept intact)"]

    Old -->|"Summarization call\nto LLM"| Summary["✨ Golden Summary\n(memory item)"]
    Summary -->|"Inject as memory object"| Context["📋 Active Context"]
    Recent --> Context
```

**Summary prompt must ensure:**
- No contradictions introduced
- Temporal ordering preserved
- No hallucinations
- Captures: device info, issues, steps tried, outcomes, current status, next steps

---

### Trimming vs. Compaction vs. Summarization

| Dimension | Trimming | Compaction | Summarization |
|-----------|----------|-----------|---------------|
| **What's removed** | Entire old turns | Tool results only | All old turns → compressed |
| **Info loss** | ✅ Yes | ⚠️ Partial | ❌ None (compressed) |
| **Latency** | ⚡ None | ⚡ None | 🐢 Slight (extra LLM call) |
| **Cost** | 💚 Cheapest | 💚 Cheap | 🟡 Moderate |
| **Best for** | Independent tasks / tool workflows | Tool-heavy agents | Dependent tasks / coaching / planning |

---

### Decision Heuristics

```mermaid
flowchart TD
    Q1{"Are tasks\nindependent\nacross turns?"}
    Q2{"Is context dominated\nby tool results?"}
    Q3{"Is every detail\ncritical for future turns?"}

    Q1 -->|Yes| Trim["✂️ Use Trimming"]
    Q1 -->|No| Q2
    Q2 -->|Yes| Compact["📦 Use Compaction"]
    Q2 -->|No| Q3
    Q3 -->|Yes| Summarize["📝 Use Summarization"]
    Q3 -->|No| Trim2["✂️ Trimming likely sufficient"]
```

**Proactive thresholds (don't wait for limits to hit):**
- 🟡 At **40%** of context limit → start monitoring
- 🔴 At **80%** of context limit → trigger trimming/compaction

---

## 8. Long-Term / Cross-Session Memory

### Cross-Session Flow

```mermaid
sequenceDiagram
    participant U as User
    participant A as Agent (Session 1)
    participant S as Memory Store
    participant B as Agent (Session 2)

    U->>A: Conversation about MacBook Wi-Fi issue
    A->>S: Extract & persist memory\n(device, OS, steps tried, status)
    Note over S: Stored: MacBook Pro, Sequoia,\nWi-Fi inactive, hard reset tried

    U->>B: New session starts
    B->>S: Retrieve relevant memories
    S-->>B: Inject memory into system prompt
    B->>U: "Hey! Still having Wi-Fi issues\nafter your macOS Sequoia update?" 🎯
```

### Memory Guard Rails

```
✅ DO:
  - Use precedence rules ("memory may be stale — verify with user")
  - Apply temporal tags (timestamp when memory was learned)
  - Separate global vs. session scope

❌ DON'T:
  - Store secrets or PII
  - Over-rely on memory without allowing override
  - Let stale memory override fresh user input
```

---

### Memory Scope

```mermaid
flowchart TD
    subgraph Global["🌍 Global Scope\n(persists across all sessions)"]
        G1["User prefers friendly tone"]
        G2["User lives in US"]
        G3["Always books window seat"]
    end

    subgraph Session["📅 Session Scope\n(relevant to current session only)"]
        S1["Wants window seat this flight to sleep"]
        S2["Troubleshooting MacBook Wi-Fi today"]
    end

    Session -->|"Repeated pattern detected"| Promote["⬆️ Graduate to Global Memory"]
    Promote --> Global
```

---

### Advanced Memory Techniques

| Technique | Description | Storage |
|-----------|-------------|---------|
| **Extraction** | Dedicated tool extracts key facts mid-session in JSON/structured format | DB / file |
| **State Management** | State object (current goal, status) injected into system prompt each turn | In-memory / DB |
| **Retrieval (RAG-style)** | Memories stored in vector DB; search → rank → inject at session start | Vector DB |

---

## 9. Prompt & Tool Hygiene

### System Prompt Best Practices

- ✅ **Lean, clear, well-structured** — no bloat
- ✅ **Small canonical few-shot examples** — quality over quantity
- ✅ **Explicit and direct language** — no ambiguity
- ✅ **Space for planning** — especially with reasoning models
- ❌ **Avoid overlapping instructions** — leads to conflict
- ❌ **Avoid vague or duplicate tool definitions** — causes noise

### Tool Design Rules

```mermaid
flowchart LR
    Bad["❌ Bad Tool Design"]
    Bad --> B1["Many similar tools"]
    Bad --> B2["Overlapping functions"]
    Bad --> B3["Verbose raw output (full JSON blobs)"]
    Bad --> B4["Machine-readable IDs only"]

    Good["✅ Good Tool Design"]
    Good --> G1["Small, targeted tool set"]
    Good --> G2["Clear, non-overlapping boundaries"]
    Good --> G3["High-signal, filtered output"]
    Good --> G4["Human-readable identifiers"]
```

---

## 10. Agent Context Profiles

```mermaid
quadrantChart
    title Agent Types by Context Domination
    x-axis Low Tool Usage --> High Tool Usage
    y-axis Low Conversation Depth --> High Conversation Depth
    quadrant-1 Tool + Dialogue Heavy
    quadrant-2 Conversational Concierge
    quadrant-3 Lightweight Agents
    quadrant-4 Tool-Heavy Workflows
    RAG Assistant: [0.25, 0.5]
    Coaching Agent: [0.2, 0.85]
    Task Automation: [0.8, 0.25]
    Planning Agent: [0.55, 0.75]
```

| Profile | Context Dominated By | Best Technique |
|---------|---------------------|----------------|
| **RAG-Heavy Assistant** | Retrieved knowledge + citations | Trim retrieved chunks; re-rank before inject |
| **Tool-Heavy Workflow** | Tool call payloads | Compaction; minimize tool verbosity |
| **Conversational Concierge** | Growing dialogue | Summarization; cross-session memory |

---

## 11. Memory Design Best Practices

```mermaid
flowchart TD
    Start["🚀 Design Agent Memory"]

    Start --> Q1["What does this agent need to remember?"]
    Q1 --> Def["Define 'meaningful context'\nfor your specific use case"]

    Def --> Evolve["Design dynamic memory lifecycle"]

    Evolve --> Promote["⬆️ Promote stable facts\nto long-term store"]
    Evolve --> Forget["🗑️ Forget stale / low-confidence\nor temporary info"]
    Evolve --> Merge["🔗 Clean, merge & consolidate\nconflicting memories"]

    Promote & Forget & Merge --> Iterate["🔄 Iterate & optimize thresholds"]
    Iterate --> Eval["📊 Run evals: Memory ON vs. OFF"]
```

### Shape of Memory (Complexity Spectrum)

```
Simple ─────────────────────────────────── Complex
  │                                           │
Key-value pairs    Structured JSON    Rich paragraph
"device: MacBook"  {device, OS, ...}  Full narrative summary
```

**Recommendation:** Start simple, evolve as needed.

---

## 12. Evaluating Memory Performance

```mermaid
flowchart LR
    E1["1️⃣ Run Core Evals\n(memory ON vs. OFF)\nBaseline comparison"]
    E2["2️⃣ Build Memory-Specific Evals\nSummary quality\nInjection timing\nGolden dataset ~50 examples"]
    E3["3️⃣ Find Right Heuristics\nOptimal trim N\nCompaction thresholds\nToken savings tracking"]

    E1 --> E2 --> E3
```

**Metrics to track:**
- Completeness of responses
- Task success rate
- Token savings (cost reduction)
- Summary accuracy vs. hallucination rate

---

## 13. Scaling Agent Memory Systems

```mermaid
flowchart TD
    Scale["📈 Scaling Strategy"]

    Scale --> Pilot["🧪 Pilot Program\nDeploy to subgroup first\nObserve memory evolution"]

    Pilot --> Type{"Memory approach?"}

    Type -->|"Retrieval-based"| R["🔍 Optimize Search System\n• Vector DB storage\n• Sharding\n• Embedding model tuning\n• Filter + rank pipeline"]
    Type -->|"Summarization-based"| S["💾 Optimize Storage\n• Disk persistence\n• Large memory node management\n• Text storage efficiency"]
```

### Scale Requirements by Agent Type

| Agent Type | Memory Volume | Evolution Speed | Scale Priority |
|------------|--------------|----------------|----------------|
| **Travel Concierge** | Low (seat, hotel prefs) | Slow | Storage efficiency |
| **Life Coach** | Very high (goals, history) | Fast | Retrieval speed + freshness |
| **IT Support** | Medium (device, issue history) | Moderate | Summarization quality |

---

## 14. Memory Pruning & Temporal Decay

### Temporal Tagging Strategy

```mermaid
flowchart LR
    M1["Memory: 'I like dogs'\n🕐 Learned: 3 months ago"]
    M2["Memory: 'I like cats'\n🕐 Learned: today"]

    M1 & M2 --> LLM["Agent reasoning"]
    LLM --> Out["Output: 'User currently\nprefers cats' ✅"]
```

### Decay Strategies

| Strategy | How It Works | Best For |
|----------|-------------|---------|
| **Temporal Text** | Tag memories with timestamps; newer overrides older | Fast-changing preferences |
| **Weighted Decay** | Score memories by recency; downweight old ones | Gradual preference shifts |
| **Hard Expiry** | Delete memories older than X days/sessions | Highly time-sensitive data |
| **Consolidation** | Merge contradictory memories into unified state | Long-running, complex agents |

---

## 15. Key Takeaways & Resources

### The Three Core Questions of Memory Design

```mermaid
flowchart TD
    Design["🎯 Agent Memory Design"]
    Design --> Q1["❓ WHAT should the\nagent remember?"]
    Design --> Q2["❓ HOW should it\nremember?"]
    Design --> Q3["❓ HOW should it\nforget?"]

    Q1 --> A1["Define meaningful context\nfor your use case"]
    Q2 --> A2["Choose: extract, state,\nor retrieval approach"]
    Q3 --> A3["Temporal tags, decay,\npruning thresholds"]
```

### Northstar Goal

> 🏆 **Aim for the smallest, highest-signal context possible.**

Every token in context should earn its place.

---

### 📚 Resources

| Resource | Purpose |
|----------|---------|
| **Context Engineering Cookbook** | Patterns and strategies reference |
| **Context Summarization Cookbook** | Summarization implementation guide |
| **Agents Python SDK (OpenAI)** | Build agents with custom session control |
| **Full Build Hour Repo (GitHub)** | Demo app + all linked resources |

---

### Quick Decision Flowchart

```mermaid
flowchart TD
    Start["🤔 Context getting too large?"]

    Start --> Q1{"Tasks\nindependent?"}
    Q1 -->|Yes| Q2{"Dominated by\ntool results?"}
    Q1 -->|No| Sum["📝 Summarize\n(preserve all info)"]

    Q2 -->|Yes| Q3{"Need\nstructure intact?"}
    Q2 -->|No| Trim["✂️ Trim\n(drop old turns)"]

    Q3 -->|Yes| Compact["📦 Compact\n(remove tool results only)"]
    Q3 -->|No| Trim

    Start --> Q4{"Cross-session\ncontinuity needed?"}
    Q4 -->|Yes| LT["🗂️ Long-term memory\n(extract + retrieve)"]
    Q4 -->|No| ST["🕐 Short-term only\n(in-session techniques)"]
```

---

*Notes compiled from: Introduction to Agent Memory and Context Engineering session*