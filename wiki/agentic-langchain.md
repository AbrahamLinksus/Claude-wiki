---
# LangChain & LangGraph

**Summary**: LangChain is a framework providing pre-built abstractions (chains, tools, memory, agents) for building agentic pipelines; LangGraph extends it with a state-machine layer for multi-agent orchestration.

**Pre-Req**: [[agentic-agent-loop]], [[agentic-multi-agent]]

**Sources**: Session-built knowledge base.

**Last updated**: 2026-04-18

**Tags**: #agentic

---

## What LangChain Is

LangChain is **framework plumbing** — it doesn't replace agents, it gives you pre-built abstractions for each stage of an agentic pipeline so you don't wire everything from scratch.

It sits between the raw LLM API and your application:

```
Raw Anthropic / OpenAI API
        │
   LangChain Core      ← chains, tool binding, memory, prompt templates
        │
    LangGraph           ← multi-agent orchestration, state machines
        │
  Your pipeline         ← planner → workers → reviewer → output
```

---

## Core Primitives

### Chain
A **Chain** is the basic unit: `input → LLM call (+ optional tools) → output`. Chains are composable.

With **LCEL** (LangChain Expression Language), the newer API, you compose with a pipe syntax:

```python
chain = prompt | llm | output_parser
result = chain.invoke({"goal": "summarize this doc"})
```

### Agent
An `Agent` is a chain that can call **tools** and loop until done — it implements the [[agentic-agent-loop]] internally. You bind tools to it and it decides when and how to call them.

```python
agent = create_tool_calling_agent(llm, tools, prompt)
executor = AgentExecutor(agent=agent, tools=tools)
executor.invoke({"input": "search for X and summarize"})
```

### Tool
A `Tool` wraps any Python function or API call, giving it a **name + description** the LLM can reason about:

```python
@tool
def search_web(query: str) -> str:
    """Search the web and return results."""
    return searcher.run(query)
```

### Memory
LangChain provides memory modules that persist state across agent steps or conversations — see [[agentic-memory]]:
- `ConversationBufferMemory` — keeps full message history in-context
- `VectorStoreRetriever` — semantic search over past interactions / documents
- `ConversationSummaryMemory` — compresses history to save tokens

---

## How it Maps to the Pipeline

| Pipeline stage | LangChain primitive |
|---|---|
| Goal → Planner | `LLMChain`, LCEL pipeline, structured output parser |
| Worker agents | `AgentExecutor` with tool bindings |
| Tools | `Tool`, `BaseTool`, built-in integrations (search, SQL, Python REPL) |
| Memory | `ConversationBufferMemory`, `VectorStoreRetriever` |
| Reviewer / routing | LCEL conditional branches, custom chain logic |
| Multi-agent orchestration | **LangGraph** (separate package) |

---

## LangGraph — Multi-Agent Layer

LangGraph models the multi-agent pipeline as a **state machine / DAG**:
- **Nodes** = agents or plain functions
- **Edges** = control flow between nodes
- **Conditional edges** = reviewer routing (PASS → end, FAIL → retry worker)
- **State** = a shared typed dict passed between all nodes

```python
from langgraph.graph import StateGraph

graph = StateGraph(MyState)
graph.add_node("planner", planner_agent)
graph.add_node("worker", worker_agent)
graph.add_node("reviewer", reviewer_agent)

graph.add_edge("planner", "worker")
graph.add_conditional_edges(
    "reviewer",
    route,                      # function that returns next node name
    {"pass": END, "fail": "worker"}
)
```

This maps directly to the [[agentic-multi-agent]] pipeline pattern.

### LangGraph vs LangChain Core
- **LangChain Core**: sequential / branching chains, single-agent loops
- **LangGraph**: cycles (retry loops), parallel fan-out, persistent state, human-in-the-loop checkpoints

---

## Built-in Integrations

LangChain ships with integrations for most common tools and data sources:
- **LLMs**: OpenAI, Anthropic, Google, local via Ollama
- **Vector stores**: Pinecone, Chroma, FAISS, Weaviate
- **Document loaders**: PDF, HTML, Notion, GitHub, S3
- **Tools**: Tavily search, Wikipedia, Python REPL, SQL, Bash

---

## LangChain vs. Rolling Your Own

LangChain is convenience, not magic. Everything it does you can do with raw API calls — it trades:

| LangChain | Raw API |
|---|---|
| Less boilerplate | Full control |
| Faster prototyping | No hidden behavior |
| Large ecosystem of integrations | Easier to debug |
| Opinionated abstractions | Portable, framework-free |

For small/one-off agents: raw API is often cleaner. For complex multi-agent pipelines with many tool integrations: LangGraph saves significant wiring effort.

---

## Related Pages

- [[agentic-multi-agent]] — The pipeline pattern LangGraph implements
- [[agentic-agent-loop]] — The inner loop each LangChain agent runs
- [[agentic-tool-use]] — How tools work in agentic systems
- [[agentic-memory]] — Memory types LangChain provides abstractions for
