---
# Multi-Agent Pipeline Pattern

**Summary**: How multiple specialized agents are composed into a pipeline — planner decomposes a goal, workers execute subtasks in parallel, a reviewer checks quality, and results are aggregated into a final output.

**Pre-Req**: [[agentic-agent-loop]], [[agentic-tool-use]]

**Sources**: Session-built knowledge base.

**Last updated**: 2026-04-18

**Tags**: #agentic

---

## The Pattern at a Glance

```
User Goal
    │
    ▼
[Planner Agent]  ── task plan (subtask DAG)
    │
    ├──► [Worker A]  ──► result_A
    ├──► [Worker B]  ──► result_B   (parallel where deps allow)
    └──► [Worker C]  ──► result_C
              │
              ▼
    [Reviewer Agent]
      ├── PASS ──► aggregate
      └── FAIL ──► back to worker (with critique) ──► retry
              │
              ▼
       Final Output
```

---

## Stage 1 — Goal Definition

The user (or upstream system) provides a **task spec**:
- What the final output should look like
- Constraints: format, tools allowed, time/cost budget
- Context the agents will need (documents, prior state)

The goal spec is the anchor for every downstream decision. Vague goals produce vague plans.

---

## Stage 2 — Planner Agent

The planner **does not execute** — it only decomposes. It outputs a structured plan:
- A list or DAG (directed acyclic graph) of **subtasks**
- Dependency edges: which steps must complete before others can start
- Per-subtask: assigned worker type, required tools, acceptance criteria

### Why a separate planner?
Separating planning from execution lets you:
- Retry failed steps without re-planning the whole task
- Run independent subtasks in parallel
- Audit the plan before execution (human-in-the-loop checkpoint)

### Planning strategies — see [[agentic-planning]]
- **Chain-of-thought decomposition** — linear list of steps
- **DAG planning** — dependency graph, parallelism-aware
- **Hierarchical planning** — coarse plan → each step expands into a sub-plan

---

## Stage 3 — Worker Agents

Each worker receives one subtask and runs its own [[agentic-agent-loop]] to complete it.

### Worker specialization
Workers are often **domain-specific** to keep context focused:

| Worker type | Tools available |
|---|---|
| Researcher | Web search, document retrieval |
| Coder | Code execution sandbox, file R/W |
| Writer | Text editor, style checker |
| Data analyst | SQL, Python, chart generation |

Specialization reduces hallucination — a coding worker doesn't see irrelevant search results.

### Parallelism
Subtasks with no dependency edges run **simultaneously**, dramatically reducing wall-clock time for complex goals.

### Worker output format
Each worker returns:
- **Result**: the artifact (text, code, data)
- **Evidence / citations**: sources used
- **Confidence signal**: optional self-assessment

---

## Stage 4 — Reviewer / Checker Agent

The reviewer is the quality gate. For each worker result it:
1. Checks result against the subtask's **acceptance criteria**
2. Looks for internal contradictions or missing requirements
3. Cross-checks consistency between results from different workers
4. Decides: **PASS** (forward to aggregation) or **FAIL** (return to worker with critique)

### Retry loop
On FAIL, the critique is appended to the worker's context and the worker retries. Most systems cap retries (e.g. 3 attempts) to prevent infinite loops.

### Why a separate reviewer?
LLMs are overconfident about their own output. A second model pass — even the same model with a different prompt — catches significantly more errors than asking the worker to self-check.

---

## Stage 5 — Final Output

Once all subtasks pass review, a **synthesis step** aggregates results:
- Merges worker outputs into a coherent whole
- Deduplicates, formats, and cites sources
- Optionally runs a final end-to-end sanity check

The synthesizer may be the planner (closing the loop) or a dedicated aggregation agent.

---

## Failure Modes

| Failure | Cause | Mitigation |
|---|---|---|
| Plan hallucination | Planner invents non-existent tools or subtasks | Validate plan against tool registry before execution |
| Worker drift | Worker solves a slightly different problem than specified | Tight acceptance criteria in subtask spec |
| Reviewer leniency | Reviewer approves bad output to avoid retries | Separate reviewer model or adversarial prompting |
| Infinite retry | Worker can't meet criteria, loops forever | Hard retry cap + escalation to human |
| Aggregation loss | Synthesis drops or contradicts valid worker results | Structured output format from workers |

---

## Related Pages

- [[agentic-agent-loop]] — The inner loop each worker runs
- [[agentic-planning]] — Planning strategies used by the planner agent
- [[agentic-tool-use]] — How workers interact with external tools
- [[agentic-evaluation]] — How to measure whether the pipeline worked
- [[agentic-safety]] — Risks specific to multi-agent systems
