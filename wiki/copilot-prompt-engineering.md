# Copilot: Prompt Engineering and Context Crafting

**Summary**: How to write effective prompts for GitHub Copilot — covering prompt structure, context determination, zero-shot vs. few-shot prompting, and engineering prompts for better performance.

**Pre-Req**: [[copilot-data-architecture]] — understanding how prompts are built helps you craft better ones.

**Sources**: https://learn.microsoft.com/credentials/certifications/resources/study-guides/gh-300, https://docs.github.com/copilot/using-github-copilot/copilot-chat/prompt-engineering-for-copilot-chat

**Last updated**: 2026-04-25

---

## Exam Weight: 10–15%

Prompt engineering is a force multiplier — better prompts improve results across every other domain (features, productivity, privacy).

---

## Prompt Structure and Context

### Anatomy of a Good Prompt
A strong Copilot prompt has four components:
1. **Goal**: what you want Copilot to produce (e.g., "write a function that…")
2. **Context**: relevant background (language, framework, constraints, existing code)
3. **Expectations**: format, style, or behavior requirements (e.g., "use async/await", "include error handling")
4. **Examples** (optional but powerful): show Copilot what good output looks like

### In Inline Suggestions
- Copilot infers the prompt from surrounding code — a good function name + a comment is often enough.
- Write a descriptive comment just above where you want the code; Copilot treats it as the goal.

### In Copilot Chat
- Be explicit: "Write a Python function that parses a CSV file and returns a list of dicts, handling missing values with None."
- Avoid vague requests like "fix this" without specifying what's wrong.

---

## How Context Is Determined

Copilot automatically gathers context from (in order of priority):
1. The **currently open file** at the cursor position
2. **Other open tabs** in the editor (neighboring files)
3. **Instructions files** (`.github/copilot-instructions.md`) — treated as persistent system context
4. **Chat history** — prior turns in the current session
5. **Repository index** — in Agent Mode, Copilot can search the repo semantically

You can *explicitly* add context by:
- Opening relevant files in tabs before prompting
- Referencing files with `#file:path` in chat
- Using `@workspace` to ask about the whole repo
- Keeping your instructions file up to date

---

## Zero-Shot and Few-Shot Prompting

### Zero-Shot
- No examples provided — just a direct instruction.
- Works well for simple, well-defined tasks.
- Example: *"Write a unit test for the `calculateTax` function."*

### Few-Shot
- Provide one or more examples of the desired input/output pattern before the actual request.
- Dramatically improves consistency for complex or unusual output formats.
- Example:
  ```
  # Input: a list of user dicts
  # Output: a markdown table with Name and Email columns
  # Example: [{"name": "Alice", "email": "a@x.com"}] → | Alice | a@x.com |
  
  Now do the same for the following list: ...
  ```
- Use few-shot prompting in chat when zero-shot gives inconsistent results.

---

## Best Practices for Prompt Crafting

| Practice | Why It Works |
|---|---|
| **Be specific and concrete** | Reduces ambiguity; LLMs perform better with well-scoped tasks |
| **State constraints explicitly** | e.g., "no external libraries", "Python 3.11+", "O(n) time complexity" |
| **Use the active file as context** | Copilot reads surrounding code — good naming and existing patterns guide the suggestion |
| **Break large tasks into steps** | Ask for one function at a time rather than an entire module |
| **Iterate** | If the first suggestion is off, refine the prompt rather than accepting a bad output |
| **Use slash commands** | `/explain`, `/fix`, `/tests`, `/doc` are faster and more targeted than free-form requests |

---

## Prompt Engineering Principles

- **Clarity > brevity**: a longer, clearer prompt outperforms a short vague one every time.
- **Role prompting**: start chat with "You are a security reviewer — identify vulnerabilities in this code." Setting a role biases the model's behavior.
- **Constraint layering**: add constraints incrementally ("make it async" → "add error handling" → "add type annotations") rather than all at once.
- **Ground in the codebase**: reference actual file names, function names, and variable names from the project — Copilot uses these as anchors.

---

## Prompt Process Flow and Chat History Usage

### Flow
```
User writes prompt
      ↓
IDE/Chat assembles: system instructions + chat history + user prompt + file context
      ↓
Full assembled prompt sent to model (see [[copilot-data-architecture]])
      ↓
Model generates response
      ↓
Response displayed; added to chat history for next turn
```

### Chat History Considerations
- Earlier turns stay in context until the window fills — older turns drop off silently.
- If you notice Copilot "forgetting" earlier instructions, the context window has likely been exceeded.
- Workaround: re-state key constraints at the start of a new session, or use an instructions file to persist them.
- **Prompt file reuse**: save reusable system prompts in `.github/copilot-instructions.md` so they apply automatically every session.
