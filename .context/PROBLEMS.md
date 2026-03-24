# AgentFabric — Problem Space

## One-Line Mission
> **Making LLM inference efficient for agentic, stateful, multi-step AI systems.**

---

## Core Problem Statement

Modern LLM inference systems are optimized for stateless, single-request workloads and fail to efficiently support agentic, multi-step workflows.

This leads to poor GPU utilization, redundant computation due to repeated prompt processing, inefficient KV cache usage, and suboptimal scheduling under heterogeneous workloads.

AgentFabric addresses these challenges by introducing session-aware execution, intelligent workload scheduling, and persistent memory management to enable efficient, scalable inference for agent-based AI systems.

---

## Problem Breakdown

### 1. Stateless Systems vs Stateful Agents
Current inference systems assume `User → LLM → Response (done)`.  
Agentic workflows are `User → LLM → Tool → LLM → Tool → LLM → ...`  
**Fundamental mismatch.**

---

### 2. KV Cache Inefficiency
KV cache is treated as request-scoped memory — discarded after every step.  
Every agent step recomputes the full context from scratch.  
**Massive redundant computation + GPU memory waste.**

---

### 3. Poor GPU Utilization
Uneven request sizes, agent pauses during tool calls, and bad batching leave GPUs idle.  
**Idle GPU = money burned.**

---

### 4. Naive Scheduling
Current schedulers use FIFO or basic batching with no awareness of:
- Session context
- Step type (prefill vs decode)
- Priority
- Agent workflow dependencies

**Biggest opportunity for improvement.**

---

### 5. Batching Breaks in Agentic Flows
Requests arrive at different times, have different sizes, and tool calls introduce gaps.  
Dynamic batching strategies fall apart under these conditions.

---

### 6. Repeated Prefill Cost
Each agent step reprocesses the same growing context.  
Prefill is expensive — doing it repeatedly is a significant latency and compute overhead.

---

### 7. No Session Abstraction
Without a session layer, there's no way to track, pause, resume, or optimize multi-step workflows.  
**Everything becomes stateless chaos.**

---

### 8. No Cost-Aware Compute Allocation
Simple queries get over-allocated resources. Heavy reasoning tasks get under-allocated.  
**Waste on both ends.**

---

## AgentFabric's Answer

| Problem | Solution |
|---|---|
| Stateless systems | Session-aware execution |
| KV cache waste | Persistent KV memory management |
| Poor scheduling | Workload-type-aware intelligent scheduler |
| GPU underutilization | Optimized batching for async agent flows |
| Repeated prefill | Context reuse across steps |
| No cost awareness | Dynamic compute allocation |
