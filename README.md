# 🔱 Hermes Multi-Agent System

<div align="center">

[![Hermes Agent](https://img.shields.io/badge/Hermes_Agent-NousResearch-purple?style=for-the-badge)](https://github.com/NousResearch/hermes-agent)
[![OpenCode AI](https://img.shields.io/badge/OpenCode-AI-blue?style=for-the-badge)](https://opencode.ai)
[![Ollama Cloud](https://img.shields.io/badge/Models-Ollama_Cloud-green?style=for-the-badge)](https://ollama.com)
[![Agents](https://img.shields.io/badge/Agents-17-green?style=for-the-badge)]()
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)]()

</div>

---

## 📋 Table of Contents

- 🏠 [Overview](#overview) — What is Hermes, compatibility, platforms
- 🏗️ [Architecture](#architecture) — System diagram and agent groups
- 🧠 [Model Selection Rationale](#model-selection-rationale) — Why each model was chosen
- 🤖 [Agents](#agents) — All 17 agents with roles and models
- 🔄 [Pipelines](#pipelines) — Predefined workflows for common tasks
- 💰 [Cost Optimization](#cost-optimization) — How models minimize cost per pipeline
- ⭐ [Key Features](#key-features) — Quality gates, security, checkpoints, learning
- 📦 [Installation](#installation) — Setup for Hermes Agent and OpenCode AI
- 🚀 [Usage](#usage) — How to run pipelines and examples
- ⚙️ [Configuration](#configuration) — Customizing models and providers
- 📁 [Project Structure](#project-structure) — File layout
- 📊 [Available Models](#available-models) — All Ollama Cloud models with specs
- 🔗 [Links](#links) — Hermes Agent, OpenCode AI, Ollama
- 🤝 [Contributing](#contributing) — How to contribute
- 📄 [License](#license) — MIT

---

## Overview

Hermes is a sophisticated multi-agent orchestration system that coordinates 17 specialized AI agents to handle complex software development tasks — from research and planning to implementation, testing, and deployment.

This system works with **both [Hermes Agent](https://github.com/NousResearch/hermes-agent)** (NousResearch's open-source agent framework) and **[OpenCode AI](https://opencode.ai)**. It uses **Ollama Cloud models**, hand-picked for each role: heavyweight models (Kimi K2.6, Cogito 671B) for critical reasoning, specialized models (Qwen3 Coder, GLM 5.1) for code writing, and lightweight models (Gemma 4 31B) for fast mechanical tasks.

### Compatibility

| Platform | How to Use |
|----------|-----------|
| **[Hermes Agent](https://github.com/NousResearch/hermes-agent)** | Use with `delegate_task` or spawn via `hermes chat -q`. Copy agent prompts to `~/.hermes/skills/` or load via `/skill`. |
| **[OpenCode AI](https://opencode.ai)** | Copy `opencode.json` and `agent/` to `~/.config/opencode/`. Agents appear as available subagents. |

Both platforms support the same core concept: a primary orchestrator that delegates to specialized subagents via `task` calls. The agent prompt files (`.md`) are the same regardless of platform — only the routing config format differs.

---

## Architecture

```
                              ┌─────────────────────────────────────┐
                              │           🔱 HERMES                 │
                              │      Master Orchestrator            │
                              │      (kimi-k2.6:cloud)              │
                              └─────────────────┬───────────────────┘
                                                │
        ┌───────────────────────────────────────┼───────────────────────────────────────┐
        │                                       │                                       │
        ▼                                       ▼                                       ▼
┌───────────────┐                     ┌───────────────┐                       ┌───────────────┐
│   RESEARCH    │                     │   PLANNING    │                       │IMPLEMENTATION │
├───────────────┤                     ├───────────────┤                       ├───────────────┤
│ @finder       │                     │ @architect    │                       │ @coder        │
│  gemma4:31b   │                     │  kimi-k2      │                       │  glm-5.1      │
│ @analyst      │                     │  -thinking    │                       │ @editor        │
│  deepseek-v3.2│                     │ @planner      │                       │  qwen3-coder   │
│ @researcher   │                     │  gemma4:31b   │                       │ @fixer        │
│  cogito-671b  │                     └───────────────┘                       │  deepseek-v3.2│
└───────────────┘                                                              │ @refactorer   │
                                                                               │  deepseek-v3.2│
                                                                               └───────────────┘
        ┌───────────────────────────────────────┼───────────────────────────────────────┐
        │                                       │                                       │
        ▼                                       ▼                                       ▼
┌───────────────┐                     ┌───────────────┐                       ┌───────────────┐
│    QUALITY    │                     │ DOCUMENTATION │                       │INFRASTRUCTURE │
├───────────────┤                     ├───────────────┤                       ├───────────────┤
│ @reviewer     │                     │ @documenter   │                       │ @devops       │
│  kimi-k2.6   │                     │  minimax-m2.7 │                       │  glm-5.1      │
│ @tester       │                     │ @commenter    │                       │ @optimizer    │
│  qwen3-coder  │                     │  gemma4:31b   │                       │  deepseek-v3.2│
│ @debugger     │                     └───────────────┘                       └───────────────┘
│  cogito-671b  │
│ @security     │
│  kimi-k2.5   │
└───────────────┘
```

---

## Model Selection Rationale

| Model | Agents | Why |
|-------|--------|-----|
| **kimi-k2.6:cloud** | hermes, reviewer | Best general reasoning. Hermes needs top-tier routing; reviewer must not miss bugs. |
| **kimi-k2-thinking:cloud** | architect | Step-by-step thinking for system design. |
| **kimi-k2.5:cloud** | security | Careful analysis for security audits. |
| **cogito-2.1:671b-cloud** | researcher, debugger | 671B MoE — massive reasoning capacity for connecting diverse info and tracing root causes. |
| **deepseek-v3.2:cloud** | analyst, fixer, refactorer, optimizer | Strong coding + deep understanding. Ideal for code analysis and targeted changes. |
| **glm-5.1:cloud** | coder, devops | Excellent at code generation and deployment/infra tasks. |
| **qwen3-coder-next:cloud** | editor, tester | Specialized coding model for modifications and test writing. |
| **minimax-m2.7:cloud** | documenter | Good prose — docs don't need the most expensive model. |
| **gemma4:31b-cloud** | finder, planner, commenter | Fast and capable for mechanical tasks (search, decompose, comment). |

---

## Agents

### 🔱 Hermes — Master Orchestrator
- **Model:** `ollama/kimi-k2.6:cloud`
- **Role:** Routes requests, manages pipelines, coordinates agents, handles conflicts
- **Tools:** `task`, `todowrite`, `todoread`

### 🔍 Research Agents

| Agent | Model | Role |
|-------|-------|------|
| **@finder** | `gemma4:31b-cloud` | Fast codebase scout. Finds files, patterns, project structure |
| **@analyst** | `deepseek-v3.2:cloud` | Deep code analysis. Dependencies, risks, data flow |
| **@researcher** | `cogito-2.1:671b-cloud` | External knowledge. Documentation, best practices, solutions |

### 📐 Planning Agents

| Agent | Model | Role |
|-------|-------|------|
| **@architect** | `kimi-k2-thinking:cloud` | Solution design. Components, interfaces, integration |
| **@planner** | `gemma4:31b-cloud` | Task decomposition. Atomic tasks, dependencies, ordering |

### 💻 Implementation Agents

| Agent | Model | Role |
|-------|-------|------|
| **@coder** | `glm-5.1:cloud` | Creates new code. Files, functions, classes |
| **@editor** | `qwen3-coder-next:cloud` | Modifies existing code. Safe changes, backward compatibility |
| **@fixer** | `deepseek-v3.2:cloud` | Fixes bugs. Minimal changes, root cause fixes |
| **@refactorer** | `deepseek-v3.2:cloud` | Improves structure. Same behavior, better code |

### ✅ Quality Agents

| Agent | Model | Role |
|-------|-------|------|
| **@reviewer** | `kimi-k2.6:cloud` | Code review. Quality, best practices, issues |
| **@tester** | `qwen3-coder-next:cloud` | Testing. Unit tests, integration tests, coverage |
| **@debugger** | `cogito-2.1:671b-cloud` | Bug investigation. Root cause analysis, diagnosis |
| **@security** | `kimi-k2.5:cloud` | Security audit. Vulnerabilities, compliance, OWASP |

### 📚 Documentation Agents

| Agent | Model | Role |
|-------|-------|------|
| **@documenter** | `minimax-m2.7:cloud` | Technical docs. README, API docs, guides |
| **@commenter** | `gemma4:31b-cloud` | Code comments. JSDoc, inline comments |

### 🔧 Infrastructure Agents

| Agent | Model | Role |
|-------|-------|------|
| **@devops** | `glm-5.1:cloud` | CI/CD, Docker, deployment, infrastructure |
| **@optimizer** | `deepseek-v3.2:cloud` | Performance. Bottlenecks, memory, efficiency |

---

## Pipelines

### New Feature
```
@finder(gemma4) → @analyst(deepseek) → @architect(kimi-thinking) → @planner(gemma4) → @coder(glm5.1) → @reviewer(kimi-k2.6) → @tester(qwen3-coder) → @documenter(minimax)
```

### New Feature (Security-Related)
```
@finder → @analyst → @researcher(cogito) → @architect → @planner → @coder → @reviewer → @security(kimi-k2.5) → @tester → @documenter
```

### Bug Fix (Unknown Cause)
```
@finder → @debugger(cogito) → @fixer(deepseek) → @reviewer → @tester
```

### Bug Fix (Known Cause)
```
@finder → @fixer(deepseek) → @reviewer → @tester
```

### Refactoring
```
@finder → @analyst → @refactorer(deepseek) → @reviewer → @tester
```

### Performance Optimization
```
@finder → @analyst → @optimizer(deepseek) → @reviewer → @tester
```

### Infrastructure Changes
```
@finder → @devops(glm5.1) → @reviewer → @tester
```

---

## Cost Optimization

Models are selected to minimize cost per pipeline. A typical "New Feature" pipeline uses:

| Step | Model | Cost Tier |
|------|-------|-----------|
| finder | gemma4:31b | $ |
| analyst | deepseek-v3.2 | $$ |
| architect | kimi-k2-thinking | $$$ |
| planner | gemma4:31b | $ |
| coder | glm-5.1 | $$ |
| reviewer | kimi-k2.6 | $$$ |
| tester | qwen3-coder | $$ |
| documenter | minimax-m2.7 | $ |

Heavy models (kimi, cogito) only run where their reasoning capacity makes a real difference. Lightweight models (gemma4, minimax) handle mechanical tasks at fraction of the cost.

---

## Key Features

- **Mandatory Quality Gates:** `@reviewer` and `@tester` run after every code change
- **Security First:** `@security` is mandatory for auth, user data, secrets
- **Checkpoints:** User confirmation required between phases
- **Revision Loops:** Up to 3 iterations for fixes before escalation
- **Context Passing:** Full context flows between all agents
- **Session Learning:** System learns from repeated issues
- **Conflict Resolution:** Priority-based resolution (security > quality > implementation)
- **Cost-Optimized Models:** Right-sized model for each task

---

## Installation

### With Hermes Agent

1. Install [Hermes Agent](https://github.com/NousResearch/hermes-agent):
```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

2. Clone this repository:
```bash
git clone https://github.com/anibalardid/hermes-multiagent.git
cd hermes-multiagent
```

3. Copy agent prompts as skills:
```bash
# Copy each agent as a Hermes skill
cp -r agent/core/hermes.md ~/.hermes/skills/autonomous-ai-agents/hermes-multiagent/hermes.md
cp -r agent/subagents/ ~/.hermes/skills/autonomous-ai-agents/hermes-multiagent/subagents/
```

4. Or load on demand:
```bash
# In a Hermes chat session:
/skill hermes-multiagent/hermes
```

5. Set Ollama Cloud as your model provider:
```bash
hermes model  # Select Ollama Cloud
# Or set in config:
hermes config set model.provider ollama
hermes config set model.base_url https://cloud.ollama.com/v1
```

### With OpenCode AI

1. Install [OpenCode AI](https://opencode.ai):
```bash
npm i -g opencode-ai@latest
# or
brew install anomalyco/tap/opencode
```

2. Clone this repository:
```bash
git clone https://github.com/anibalardid/hermes-multiagent.git
cd hermes-multiagent
```

3. Install dependencies:
```bash
bun install
# or
npm install
```

4. Copy agent files to OpenCode config:
```bash
cp opencode.json ~/.config/opencode/opencode.json
cp -r agent ~/.config/opencode/agent
```

5. Set your Ollama Cloud API key:
```bash
export OLLAMA_CLOUD_KEY="your-api-key-here"
# Or login via OpenCode:
opencode auth login
```

### Verify Models

```bash
ollama list
```

---

## Usage

### With Hermes Agent

```bash
# Start an interactive session
hermes

# Or one-shot with the orchestrator skill
hermes -s hermes-multiagent/hermes
```

Hermes will automatically route your request to the appropriate pipeline.

### With OpenCode AI

The system activates automatically when you use OpenCode AI. Hermes analyzes your request and routes it to the appropriate pipeline.

**Examples:**

```
"Add user authentication with JWT"
→ Security-related feature pipeline

"Fix the login bug"
→ Bug fix pipeline (debugger if cause unknown)

"Refactor the UserService"
→ Refactoring pipeline

"Optimize database queries"
→ Performance pipeline

"Set up Docker and CI/CD"
→ Infrastructure pipeline
```

---

## Configuration

### For OpenCode AI

All configuration is in `opencode.json`:

- **Models:** Customize models for each agent (swap models based on availability/cost)
- **Tools:** Enable/disable tools per agent
- **Provider:** Ollama Cloud endpoint and API key
- **LSP:** Language Server Protocol integration

### For Hermes Agent

Configure in `~/.hermes/config.yaml`:

```yaml
model:
  provider: ollama
  base_url: https://cloud.ollama.com/v1
  api_key: ${OLLAMA_CLOUD_KEY}

delegation:
  model: ollama/kimi-k2.6:cloud
  max_iterations: 50
```

### Switching Models

To use a different model for an agent, edit `opencode.json`:

```json
"coder": {
  "model": "ollama/deepseek-v3.2:cloud"
}
```

Or for Hermes Agent, specify when calling:

```bash
hermes chat -m ollama/deepseek-v3.2:cloud -q "Fix the auth bug"
```

---

## Project Structure

```
hermes-multiagent/
├── opencode.json              # OpenCode AI config (agents, providers, models)
├── package.json               # Dependencies (for OpenCode AI plugin system)
├── LICENSE                    # MIT
├── README.md                  # This file
└── agent/
    ├── core/
    │   └── hermes.md          # Master orchestrator (kimi-k2.6)
    └── subagents/
        ├── research/
        │   ├── finder.md      # Codebase scout (gemma4:31b)
        │   ├── analyst.md     # Code analyst (deepseek-v3.2)
        │   └── researcher.md  # External knowledge (cogito-671b)
        ├── planning/
        │   ├── architect.md   # Solution design (kimi-k2-thinking)
        │   └── planner.md     # Task decomposition (gemma4:31b)
        ├── implementation/
        │   ├── coder.md       # Code writer (glm-5.1)
        │   ├── editor.md      # Code editor (qwen3-coder-next)
        │   ├── fixer.md       # Bug fixer (deepseek-v3.2)
        │   └── refactorer.md  # Code refactorer (deepseek-v3.2)
        ├── quality/
        │   ├── reviewer.md    # Code reviewer (kimi-k2.6)
        │   ├── tester.md      # Test engineer (qwen3-coder-next)
        │   ├── debugger.md    # Bug investigator (cogito-671b)
        │   └── security.md    # Security auditor (kimi-k2.5)
        ├── documentation/
        │   ├── documenter.md  # Technical writer (minimax-m2.7)
        │   └── commenter.md   # Code commenter (gemma4:31b)
        └── infrastructure/
            ├── devops.md      # DevOps engineer (glm-5.1)
            └── optimizer.md  # Performance optimizer (deepseek-v3.2)
```

---

## Available Models

| Model | Context | Output | Best For |
|-------|---------|--------|----------|
| kimi-k2.6:cloud | 128K | 8K | General reasoning, orchestration |
| kimi-k2-thinking:cloud | 128K | 8K | Step-by-step reasoning, design |
| kimi-k2.5:cloud | 128K | 8K | Careful analysis, security |
| cogito-2.1:671b-cloud | 128K | 8K | Massive reasoning, research, debug |
| deepseek-v3.2:cloud | 128K | 8K | Code analysis, fixes, refactoring |
| glm-5.1:cloud | 128K | 8K | Code generation, DevOps |
| qwen3-coder-next:cloud | 128K | 8K | Specialized coding, testing |
| devstral-2:123b-cloud | 128K | 8K | DevOps/infra (alternative) |
| nemotron-3-super:cloud | 128K | 8K | Instruction following |
| minimax-m2.7:cloud | 128K | 8K | Documentation, prose |
| qwen3.5:cloud | 128K | 8K | General purpose (alternative) |
| gemma4:31b-cloud | 128K | 8K | Fast mechanical tasks |
| ministral-3:8b-cloud | 128K | 8K | Ultra-fast trivial tasks |
| ministral-3:3b-cloud | 128K | 8K | Ultra-fast trivial tasks |

---

## Links

- **Hermes Agent** — [github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) — Open-source AI agent framework by NousResearch
- **Hermes Agent Docs** — [hermes-agent.nousresearch.com](https://hermes-agent.nousresearch.com/docs/)
- **OpenCode AI** — [opencode.ai](https://opencode.ai) — Provider-agnostic AI coding agent
- **Ollama** — [ollama.com](https://ollama.com) — Run and manage LLMs locally and in the cloud

---

## Contributing

Contributions are welcome! Areas of interest:

- Better model-to-task mappings based on benchmarks
- New specialized agents (e.g., data pipeline, ML training)
- Pipeline templates for common workflows
- Cost/performance benchmarks across models
- Hermes Agent-specific skill packaging

---

## License

MIT

---

<div align="center">
  <sub>Built with ❤️ — Powered by <a href="https://github.com/NousResearch/hermes-agent">Hermes Agent</a>, <a href="https://opencode.ai">OpenCode AI</a>, and <a href="https://ollama.com">Ollama Cloud</a></sub>
</div>