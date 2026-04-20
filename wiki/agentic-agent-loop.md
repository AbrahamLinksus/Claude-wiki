---
# The Agent Loop

**Summary**: The core execution pattern of an agentic AI system — a cycle of observation, reasoning, and action that repeats until a stopping condition is met.

**Pre-Req**: Basic understanding of LLMs as next-token predictors.

**Sources**: Session-built knowledge base.

**Last updated**: 2026-04-18

**Tags**: #agentic

---

## The Basic Loop

```
while goal_not_met:
    observation = gather_context()   # current state, tool results, memory
    action      = llm.think(observation)  # plan next step
    result      = execute(action)    # call tool, write file, etc.
    observation = update(result)
```

Each turn the model sees: **system prompt + history + latest observation**, then emits either a **tool call** or a **final answer**.

---

## Key Components

### Observation / Context Window
Everything the agent can "see" at a given step: the goal, prior actions and their outputs, memory retrievals, and any injected state. The context window is the agent's working memory.

### Reasoning Step
The LLM decides *what to do next*. This may be:
- A direct tool call (structured output)
- A scratchpad reasoning trace (chain-of-thought) before a tool call — see [[agentic-planning]]
- A final answer / stopping signal

### Action
The agent does something in the world:
- Calls a tool / API — see [[agentic-tool-use]]
- Writes to memory — see [[agentic-memory]]
- Spawns a sub-agent — see [[agentic-multi-agent]]
- Returns output to the user

### Stopping Conditions
- Goal explicitly met (model emits a final answer)
- Max steps reached (hard safety cap)
- Human intervention / approval required
- Error / impossible state detected

---

## Loop Variants

| Variant | Description |
|---|---|
| **ReAct** | Alternates Thought → Action → Observation traces in-context |
| **Reflexion** | Agent critiques its own prior attempt before retrying |
| **Plan-then-execute** | Full plan generated upfront, then executed step-by-step |
| **MCTS / tree search** | Explores multiple action branches, backtracks — see [[agentic-planning]] |

---

## Failure Modes

- **Looping**: agent repeats the same action because it doesn't recognize it already tried
- **Context overflow**: long task history exceeds context window, early steps get dropped
- **Hallucinated tool calls**: model invents a tool name or argument that doesn't exist
- **No stopping signal**: model never emits a final answer, burns tokens until hard cap
