# AgentFabric

**Agent-Aware LLM Inference Fabric with Session-Based Scheduling and KV Memory Optimization**

---

## Overview

AgentFabric is a next-generation LLM inference system designed to efficiently execute **agentic, multi-step AI workflows**.

Traditional inference systems are optimized for **stateless, single-request interactions**, but modern AI systems increasingly rely on **stateful agents** that perform iterative reasoning, tool usage, and long-lived execution.

AgentFabric introduces a **compute fabric for intelligent workloads**, enabling efficient execution through:

- Session-aware scheduling
- Persistent KV memory management
- Dynamic batching across heterogeneous workloads
- Agent-native execution model

---

## Problem Statement

Modern LLM inference systems face critical limitations:

- Optimized for *single request → single response* workflows
- KV cache treated as request-scoped → no reuse across steps
- Inefficient GPU utilization under asynchronous workloads
- Naive schedulers (FIFO, static batching)
- Repeated prefill computation across agent steps
- No abstraction for session lifecycle or multi-step execution

Result: increased latency, high compute cost, poor scalability for agent-based systems.

---

## Solution

### 1. Session-Aware Execution
Treats agent workflows as first-class entities. Enables pause / resume / continuation with persistent execution state.

### 2. Intelligent Scheduling Layer
Workload-aware scheduling (prefill, decode, agent-step), dynamic batching, and priority/latency-aware dispatching.

### 3. KV Cache as Memory System
Persistent KV cache across multi-step workflows — reduces redundant computation and improves memory efficiency.

### 4. Agent-Native Runtime
Supports multi-step reasoning loops (`LLM → Tool → LLM → Tool → Response`) with async execution and tool latency handling.

### 5. Efficient Compute Utilization
Maximizes GPU utilization under mixed workloads, reduces idle compute during agent pauses.

---

## System Architecture

```
Client / Agent
      ↓
API Gateway
      ↓
Request Router
      ↓
Global Scheduler  ← Core Intelligence
      ↓
Session Manager
      ↓
Inference Engine (vLLM / custom)
      ↓
GPU Fabric
      ↓
KV Memory Layer
```

---

## Core Components

| Component | Responsibility |
|---|---|
| **Scheduler** | Dynamic batching, workload classification, GPU slot allocation |
| **Session Manager** | Tracks agent workflows, maintains state, enables pause/resume |
| **Memory Layer** | KV cache abstraction, session-level persistence, efficient reuse |
| **Router** | Model selection, request routing, cost-aware dispatch |
| **Inference Engine** | vLLM / TensorRT integration, token streaming, KV cache hooks |

---

## Key Concepts

**Prefill vs Decode Separation** — Prefill is heavy parallel computation (prompt processing); decode is sequential token generation. AgentFabric optimizes both phases independently.

**Continuous Batching** — Mixes requests at different stages to maintain high GPU utilization.

**Session-Based KV Memory** — KV cache persists across steps, avoiding recomputation of previous context.

---

## Evaluation Metrics

- Latency: p50 / p95 response time
- Throughput: tokens per second
- GPU Utilization: % active compute
- KV Cache Reuse Rate
- End-to-End Agent Execution Time
- Cost Efficiency: tokens per dollar

---

## Experiments

- Static vs dynamic batching
- Session-aware vs stateless execution
- KV cache reuse impact
- Prefill vs decode scheduling strategies
- Agent workflow performance vs standard inference

---

## Tech Stack

- **Python** — core system
- **vLLM** — inference engine
- **FastAPI** — API layer
- **Redis** — state + caching
- **Docker / Kubernetes** — deployment
- **CUDA** — optional optimization layer

---

## Roadmap

| Phase | Focus |
|---|---|
| 1 | Basic inference pipeline, API + routing layer |
| 2 | Dynamic batching scheduler, KV cache integration |
| 3 | Session-aware execution, agent workflow support |
| 4 | Advanced scheduling strategies, performance benchmarking |
| 5 | Distributed / multi-GPU simulation, research-level optimizations |

---

## Vision

> LLM inference as a compute fabric for intelligent systems — not just an API layer.

---

## Status

🚧 Work in progress — actively building core scheduling and memory systems.

---

## License

MIT
