# Copilot: Features

**Summary**: The largest exam domain — covers Copilot in the IDE, the CLI, Agent Mode, Edit Mode, MCP, Spaces, Spark, PR summaries, and organization-wide management.

**Pre-Req**: Basic familiarity with an IDE (VS Code recommended), git, and GitHub.

**Sources**: https://learn.microsoft.com/credentials/certifications/resources/study-guides/gh-300, https://docs.github.com/copilot/about-github-copilot/plans-for-github-copilot

**Last updated**: 2026-04-25

---

## Exam Weight: 25–30% (highest domain)

This is the most practical domain. It covers how to actually use Copilot day-to-day and how admins configure it at scale.

---

## Copilot in the IDE

### Enabling Copilot
- Install the GitHub Copilot extension in VS Code, JetBrains, Visual Studio, or Neovim.
- Sign in with a GitHub account that has an active Copilot plan.
- Extension activates inline suggestions automatically once installed and authenticated.

### Trigger Methods
| Method | How |
|---|---|
| **Inline suggestions** | Start typing — Copilot auto-completes; Tab to accept, Esc to dismiss |
| **Copilot Chat** | Open chat panel (Ctrl+Shift+I in VS Code); ask questions in natural language |
| **CLI** | `gh copilot` commands in terminal; see CLI section below |
| **Plan Mode** | Copilot proposes a multi-step plan before making edits; user reviews before execution |

### Excluding Files from Context
- Copilot uses open files and workspace context when generating suggestions.
- You can exclude specific files or repositories from being used as context (app knowledge).
- Configured via `.github/copilot-instructions.md` or org-level content exclusion settings.
- See [[copilot-privacy-safeguards]] for full exclusion configuration details.

---

## GitHub Copilot CLI

### What It Is
- A CLI extension for `gh` (GitHub CLI) that brings Copilot into the terminal.
- Benefits: explain shell commands, suggest commands, generate scripts, fix errors — without leaving the terminal.

### Installation
```bash
gh extension install github/gh-copilot
gh copilot --version   # verify install
```

### Key Commands
| Command | Purpose |
|---|---|
| `gh copilot suggest` | Suggest a shell command based on a description |
| `gh copilot explain` | Explain what a given command does |
| `gh copilot alias` | Set up shell aliases for quick access |

### Interactive Sessions
- Run `gh copilot suggest` interactively: Copilot asks clarifying questions and refines the suggestion.
- Supports multi-turn sessions — context carries across turns within the same session.

### Script Generation and File Management
- Describe what a script should do; Copilot generates the full shell script.
- Can generate file creation, manipulation, and cleanup scripts from natural language descriptions.

---

## Agent Mode, Edit Mode, and MCP

### Edit Mode
- Copilot edits code across one or more files simultaneously based on a natural language instruction.
- Differs from inline suggestions: you describe a change, Copilot applies it as a diff you can accept/reject.

### Agent Mode
- Copilot acts as an autonomous agent: it reads files, makes a plan, executes multi-step edits, and runs tools.
- Can invoke external tools (compilers, test runners, linters) as part of the workflow.
- **Agent Sessions**: a persistent context that tracks what the agent has done across multiple steps.
- **Sub-Agents**: delegate specific tasks to sub-agents to optimize context usage and keep the main agent focused.

### MCP (Model Context Protocol)
- An open protocol that lets Copilot connect to external context sources (databases, APIs, documentation, internal tools).
- Copilot in Agent Mode can use MCP servers to pull in live context beyond what's in the local workspace.
- Enables enhanced development workflows that integrate with third-party services.

---

## Copilot for Code Review and Coding Assistance

- **Code review**: Copilot can review a diff or PR and suggest improvements, flag potential bugs, and note style issues.
- **Coding assistance**: autocomplete, docstring generation, function stubs, type annotations, test skeletons.
- **PR Summaries**: Copilot automatically generates a summary of a pull request based on the diff and commit messages.
- **Customizable review standards**: define review expectations via instructions files (`.github/copilot-instructions.md`), so Copilot applies your team's standards consistently.

---

## Spaces and Spark

- **Spaces**: a collaborative environment in GitHub where Copilot can assist across a shared context (repositories, docs, issues).
- **Spark**: a feature for quickly generating and iterating on ideas or prototypes using Copilot — focused on rapid exploration.
- Both extend Copilot beyond the IDE into the broader GitHub platform.

---

## Copilot Chat — Limits, Options, and Commands

- **Context limit**: Copilot Chat maintains a rolling chat history; older turns are dropped as context fills up.
- **Slash commands** (in chat): e.g., `/explain`, `/fix`, `/tests`, `/doc` — trigger specific Copilot behaviors quickly.
- **Prompt file reuse**: save prompt files (`.github/copilot-instructions.md`) to reuse consistent instructions across chat sessions without retyping them.
- **Feedback**: thumbs up/down on suggestions feeds back to GitHub for model improvement.

---

## Organization-Wide Settings and Policies

### Policy Management
- Admins configure Copilot features at the organization or enterprise level via GitHub settings.
- Can **enable or disable specific features** (e.g., Copilot Chat, CLI, code review) per IDE and per github.com.
- **Copilot Code Review policies**: enforce review standards across all repos in the org.

### Audit Logs
- GitHub logs Copilot-related events (enablement, policy changes, seat assignments) in the audit log.
- Useful for compliance: you can verify who changed what and when.

### Managing Subscriptions via REST API
- GitHub's REST API exposes endpoints to list, add, and remove Copilot seats programmatically.
- Useful for automating onboarding/offboarding in large orgs.

See also [[copilot-privacy-safeguards]] for content exclusion configuration at the org level.
