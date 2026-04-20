---
# Agentic AI — Overview

**Summary**: A topic map covering the key concepts behind agentic AI systems — agents that plan, use tools, and act autonomously to complete multi-step goals.

**Sources**: Session-built knowledge base (no raw source yet).

**Last updated**: 2026-04-18

**Tags**: #agentic

---

## What is Agentic AI?

An **agent** is an LLM embedded in a loop where it can observe state, decide on actions, execute tools, and iterate until a goal is met — rather than producing a single response and stopping.

---

## Core Concept Pages

- [[agentic-agent-loop]] — The observe → think → act cycle; how agents run iteratively
- [[agentic-tool-use]] — How agents call external tools (functions, APIs, code execution)
- [[agentic-planning]] — Planning strategies: chain-of-thought, ReAct, tree-of-thought, task decomposition
- [[agentic-memory]] — Memory types: in-context, external (vector DB, key-value), episodic, semantic
- [[agentic-multi-agent]] — Multi-agent architectures: orchestrators, sub-agents, swarms, debate patterns
- [[agentic-evaluation]] — How to evaluate agent systems: task success, trajectory quality, cost/latency tradeoffs
- [[agentic-safety]] — Alignment concerns specific to agents: reward hacking, irreversible actions, human-in-the-loop

---

## Roadmap

Suggested reading order for building up a full mental model:

1. [[agentic-agent-loop]] — understand the core loop first
2. [[agentic-tool-use]] — agents are useless without tools
3. [[agentic-planning]] — how agents decide what to do next
4. [[agentic-memory]] — how agents persist state across steps
5. [[agentic-multi-agent]] — composing agents into systems
6. [[agentic-evaluation]] — measuring whether it works
7. [[agentic-safety]] — what can go wrong

---

> Pages in this cluster are added incrementally as the session progresses.
