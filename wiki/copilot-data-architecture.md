# Copilot: Data and Architecture

**Summary**: How GitHub Copilot processes your code under the hood — data flow, prompt construction, proxy filtering, the code suggestion lifecycle, and the inherent limits of LLMs.

**Pre-Req**: Basic understanding of what an LLM is; familiarity with [[copilot-responsible-ai]].

**Sources**: https://learn.microsoft.com/credentials/certifications/resources/study-guides/gh-300, https://github.com/trust-center

**Last updated**: 2026-04-25

---

## Exam Weight: 10–15%

More conceptual than the features domain — the exam tests whether you understand *what happens* to your code when Copilot processes it, not just how to use it.

---

## Data Usage, Flow, and Sharing

### What Data Copilot Sends
- When you trigger a suggestion, your IDE sends a **prompt** to GitHub's Copilot proxy — this includes surrounding code context (open files, cursor position, adjacent code).
- **Copilot Individual**: GitHub does not use prompts or suggestions to train models, and does not retain them beyond the session by default.
- **Copilot Business / Enterprise**: prompts are not used for training; stricter data retention controls are available.
- Data is transmitted over HTTPS to GitHub's backend infrastructure.

### What Is NOT Sent
- Files explicitly excluded via content exclusion settings (see [[copilot-privacy-safeguards]]).
- Code outside the active context window.

---

## Input Processing and Prompt Building

### How the Prompt Is Assembled
1. **Cursor context**: code immediately before and after the cursor position.
2. **Open tabs**: content from other open files in the editor (neighboring context).
3. **Repository context**: in some modes, Copilot indexes the wider repo for semantic search.
4. **Chat history**: in Copilot Chat, prior turns are included until the context window fills.
5. **Instructions files**: content from `.github/copilot-instructions.md` is prepended as system context.

The assembled prompt is sent to the model. This is why [[copilot-prompt-engineering]] matters — better context in = better suggestions out.

### Context Window
- The model has a fixed token budget. If the assembled context exceeds it, older content is dropped (chat history is trimmed first).
- This is a key limitation: Copilot cannot "see" your entire codebase at once.

---

## Proxy Filtering and Post-Processing

### GitHub Copilot Proxy
- All requests pass through GitHub's **proxy service** before reaching the underlying model.
- The proxy enforces:
  - **Content exclusions**: strips excluded file content from prompts.
  - **Safety filters**: blocks suggestions that contain certain harmful patterns.
  - **Duplication detection**: flags suggestions that closely match public training data (when enabled).

### Post-Processing
- After the model returns a response, GitHub applies additional filters:
  - **Security vulnerability filtering**: suggestions containing known insecure patterns (e.g., hardcoded secrets, obvious injection) may be suppressed.
  - **License/duplication check**: if enabled, suggestions matching public code with restrictive licenses are flagged or blocked.
- The filtered suggestion is then returned to the IDE.

---

## Code Suggestion Lifecycle

```
User types code in IDE
        ↓
IDE assembles prompt (cursor context + open tabs + instructions)
        ↓
Prompt sent to GitHub Copilot Proxy (HTTPS)
        ↓
Proxy applies content exclusions + pre-filters
        ↓
Filtered prompt sent to LLM (Copilot model)
        ↓
Model generates candidate completions
        ↓
Post-processing: safety filter + duplication check
        ↓
Suggestion returned to IDE
        ↓
Developer accepts (Tab) or dismisses (Esc)
```

---

## Limitations of LLMs and Copilot

| Limitation | Implication |
|---|---|
| **Fixed context window** | Cannot see your whole codebase; misses cross-file dependencies |
| **No runtime awareness** | Cannot run your code; doesn't know what it actually does at runtime |
| **Hallucinations** | May generate plausible but incorrect API calls, types, or logic |
| **Training data cutoff** | Unaware of libraries or APIs released after training cutoff |
| **Non-determinism** | Same prompt may yield different suggestions on different calls |
| **No persistent memory** | Each session starts fresh; no long-term memory of your project |
| **Statistical, not logical** | Generates based on pattern similarity, not semantic correctness |

These limitations directly motivate the responsible AI practices in [[copilot-responsible-ai]] and the prompt engineering strategies in [[copilot-prompt-engineering]].
