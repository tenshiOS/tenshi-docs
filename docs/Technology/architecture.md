# Architecture

<small><em>A pragmatic, auditable service mesh</em></small>

---



```
Client (Web/Telegram) 
   └─> API Gateway (Node)
         ├─ Auth & Token‑Gate (wallet signature + $TENSHI threshold)
         ├─ Research Service  ─┬─ Price/Pair Index (read-only)
         │                     └─ The Graph Subgraph Client
         ├─ Simulation Service (constant product / CL approximations)
         └─ LLM Orchestrator (LangGraph/LangChain tools)
```
- **LLM Core**: hosted multimodal LLM for reasoning + summarization.  
- **Cache**: Redis (30–120s) for hot queries.  
- **Observability**: structured logs + response hashes.
