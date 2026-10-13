# AgentFabric

**Design for an agent-aware LLM inference layer: session-based scheduling and KV-cache reuse for multi-step agent workflows.**

> **Status: design stage.** This repository currently holds the design write-up (this README) and a problem
> analysis ([`.context/PROBLEMS.md`](.context/PROBLEMS.md)). There is no implementation code yet; everything
> below describes the planned system.

---

## Motivation

LLM serving stacks are built around *request in, response out*. Agentic workloads look different:

```
User → LLM → Tool → LLM → Tool → LLM → ... → Response
```

Each step re-sends a growing context, tool calls leave the model idle for unpredictable periods, and requests
from many agents arrive with very different sizes. The problems AgentFabric targets:

- **No session abstraction** – the server cannot tell that ten requests belong to one agent run, so it cannot
  track, pause, resume or prioritise the workflow as a unit.
- **Repeated prefill** – every step reprocesses the shared context. Prefix caching in engines such as vLLM and
  SGLang helps, but it is best-effort and evicted under memory pressure with no knowledge of which sessions are
  about to resume.
- **KV memory not managed per session** – there is no policy for keeping, offloading or dropping a paused
  agent's KV cache while it waits on a tool.
- **Workload-blind scheduling** – FIFO or plain continuous batching ignores step type (prefill vs decode),
  session priority and workflow dependencies.
- **Uneven GPU utilisation** – tool-call gaps and mixed request sizes leave capacity idle.

See [`.context/PROBLEMS.md`](.context/PROBLEMS.md) for the full breakdown.

## Planned approach

1. **Session-aware execution** – agent workflows are first-class sessions with pause / resume / continue.
2. **Workload-aware scheduler** – classifies work as prefill, decode or agent step; dynamic batching with
   priority- and latency-aware dispatch.
3. **KV cache as a memory tier** – session-scoped KV persistence and reuse across steps, with eviction /
   offload policies informed by session state.
4. **Agent-native runtime** – async handling of `LLM → Tool → LLM` loops, including tool-call latency.
5. **Cost-aware allocation** – route light and heavy requests to appropriately sized compute.

## Planned architecture

```
Client / Agent
      ↓
API Gateway
      ↓
Request Router        – model selection, cost-aware dispatch
      ↓
Global Scheduler      – workload classification, batching, GPU slot allocation
      ↓
Session Manager       – workflow state, pause / resume
      ↓
Inference Engine      – vLLM (or TensorRT-LLM) with KV-cache hooks, token streaming
      ↓
KV Memory Layer       – session-level KV persistence and reuse
```

Intended stack: Python, vLLM, FastAPI, Redis (session state), Docker / Kubernetes; CUDA only for optional
optimisations.

## Evaluation plan

Metrics: p50 / p95 latency, tokens/s throughput, GPU utilisation, KV-cache reuse rate, end-to-end agent run
time, tokens per dollar.

Planned experiments:

- static vs dynamic batching
- session-aware vs stateless execution
- impact of KV-cache reuse
- prefill / decode scheduling strategies
- agent workflow performance vs a standard inference server

No results yet.

## Roadmap

| Phase | Focus |
|---|---|
| 1 | Basic inference pipeline, API and routing layer |
| 2 | Dynamic batching scheduler, KV-cache integration |
| 3 | Session-aware execution, agent workflow support |
| 4 | Advanced scheduling strategies, benchmarking |
| 5 | Distributed / multi-GPU experiments |

## Related work

- Orca: continuous (iteration-level) batching – Yu et al., OSDI 2022
- vLLM / PagedAttention – Kwon et al., SOSP 2023
- SGLang / RadixAttention prefix sharing – Zheng et al., NeurIPS 2024
- DistServe: prefill/decode disaggregation – Zhong et al., OSDI 2024
- InferCept: KV handling during tool-call interruptions – Abhyankar et al., ICML 2024
- Parrot: application-level (multi-call) LLM serving – Lin et al., OSDI 2024
