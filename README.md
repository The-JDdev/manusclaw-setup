<div align="center">

<img src="https://img.shields.io/badge/ManusClaw-v5.0.0-ff69b4?style=for-the-badge&logo=github&logoColor=white" alt="Version">
<img src="https://img.shields.io/badge/Setup_Guide-Production_Ready-00C853?style=for-the-badge&logo=book&logoColor=white" alt="Setup Guide">
<img src="https://img.shields.io/badge/Python-3.11+-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python">
<img src="https://img.shields.io/badge/License-Modified_MIT-FFD700?style=for-the-badge&logo=opensourceinitiative&logoColor=black" alt="License">
<img src="https://img.shields.io/badge/Tests-210_Passing-00C853?style=for-the-badge&logo=check-circle&logoColor=white" alt="Tests">
<img src="https://img.shields.io/badge/Features-14%2B_Categories-6C63FF?style=for-the-badge&logo=rocket&logoColor=white" alt="Features">

<br><br>

# MANUSCLAW — THE DEFINITIVE SETUP GUIDE

### _From Zero to Autonomous AI Beast in Minutes_

**Production-grade installation and setup documentation for ManusClaw v5.0.0 — the omnichannel autonomous AI agent framework.**

<p>
  <a href="https://github.com/ManusAgents/manusclaw">
    <img src="https://img.shields.io/badge/Core_Engine-ManusAgents%2Fmanusclaw-181717?style=for-the-badge&logo=github&logoColor=white" alt="Core Engine">
  </a>
  &nbsp;
  <a href="https://github.com/The-JDdev/manusclaw-setup">
    <img src="https://img.shields.io/badge/This_Repo-Setup_%26_Docs-00C853?style=for-the-badge&logo=github&logoColor=white" alt="Setup Repo">
  </a>
  &nbsp;
  <a href="https://github.com/The-JDdev">
    <img src="https://img.shields.io/badge/Built_by-The--JDdev_(SHS_Lab)-FFFFFF?style=for-the-badge&logo=github&logoColor=181717" alt="Developer">
  </a>
</p>

---

## Repository Architecture

> **IMPORTANT — READ THIS FIRST**: The ManusClaw ecosystem is split across TWO repositories. Understanding this structure is essential for installation.

| Repository | URL | Purpose |
|---|---|---|
| **Core Engine** | [`ManusAgents/manusclaw`](https://github.com/ManusAgents/manusclaw) | Source code, agent logic, tools, LLM providers, channels, server, voice, sandbox — **everything that runs** |
| **Setup & Docs** | [`The-JDdev/manusclaw-setup`](https://github.com/The-JDdev/manusclaw-setup) (this repo) | Installation guides, configuration reference, platform-specific tutorials, deployment docs |

```
  WHAT YOU CLONE          WHERE IT LIVES         WHAT IT DOES
  ─────────────────    ────────────────────    ────────────────────────
  manusclaw/            ManusAgents/manusclaw     The AI agent engine itself
  manusclaw-setup/      The-JDdev/manusclaw-setup The installation guide you're reading
```

**In short**: You clone the **Core Engine** (`ManusAgents/manusclaw`) to run ManusClaw. This repository (`manusclaw-setup`) exists to guide you through that process with production-grade documentation.

---

---

<br>

```
     /\___/\
    (  o o  )
    (  =^=  )     PAORR Loop: Plan -> Act -> Observe -> Reflect -> Retry
     (     )      12+ Channels | Voice | Canvas | SSH | Webhooks | Multi-Agent
      \___/       10+ LLM Providers | 3 Sandboxes | 210 Tests | Production-Ready
           v5.0.0
```

</div>

---

---

## Table of Contents

| # | Section | Description |
|---|---|---|
| 1 | [Framework Introduction](#1--framework-introduction) | Architecture, PAORR loop, design philosophy, what makes ManusClaw different |
| 2 | [Prerequisites](#2--prerequisites) | System requirements, API keys, hardware, platform compatibility |
| 3 | [Installation Guide](#3--installation-guide) | pip, Docker, Termux, Colab, WSL2, installer scripts, dependency groups |
| 4 | [Configuration Guide](#4--configuration-guide) | config.toml, config.yaml, .env, priority chain, all 60+ env vars, profiles |
| 5 | [Running Modes](#5--running-modes) | CLI, server, multi-agent, cron, voice, SSH, channels, webhooks — 12 entry points |
| 6 | [Feature Showcase](#6--feature-showcase) | All 14+ v5 features with setup instructions, config, and usage examples |
| 7 | [Security & Memory](#7--security--memory) | Identity Guard, Permission Gate, Credential Pool, Secret Redaction, 3-Tier Memory |

> **Quick links to sub-guides**: [Installation](docs/installation.md) | [Configuration](docs/configuration.md) | [Usage](docs/usage.md) | [Features](docs/features.md) | [Deployment](docs/deployment.md) | [Troubleshooting](docs/troubleshooting.md) | [Platforms](docs/platforms/)

---

<br>

---

## 1 — Framework Introduction

### What is ManusClaw?

ManusClaw is **Bangladesh's first production-grade autonomous AI agent framework** — a Python 3.11+ system engineered for real-world task completion across multiple channels. It does not simply generate text; it **plans, acts, observes, reflects, and retries** through a sophisticated reasoning loop until the objective is achieved. Whether you need to automate code review, manage email, browse the web, deploy infrastructure, or coordinate multi-agent workflows — ManusClaw handles it autonomously.

Unlike simple LLM wrappers or chatbot frameworks, ManusClaw operates as a **full autonomous agent** with access to 14+ tools, persistent memory, 10+ LLM provider integrations, and communication across 12+ messaging platforms simultaneously. In v5.0.0, ManusClaw evolved from a CLI-only tool into a complete **omnichannel AI platform** with voice interaction, live UI rendering via Canvas, SSH remote control, webhook-driven automation, Gmail integration, and multi-agent orchestration.

The framework is designed around one core principle: **the agent should be able to do anything a skilled developer can do on a terminal, but autonomously, reliably, and across any communication channel the user prefers.**

### Architecture Overview

ManusClaw's architecture follows a layered design pattern with clear separation between concerns. At the highest level, users interact through one of 12 entry points — CLI shell, HTTP/WS server, cron scheduler, SSH gateway, voice wake word, or any of the 12 messaging channels. Each entry point funnels into the **Agent Layer**, which contains the PAORR reasoning loop, tool dispatch, memory management, and security guards. The Agent Layer communicates with the **LLM Layer**, which handles model routing, credential rotation, adaptive timeouts, and automatic failover across 10+ providers. Below these layers, the **Integration Layer** manages external connections: messaging adapters, Canvas A2UI protocol, webhook HMAC verification, mobile/desktop nodes, and sandboxed code execution across three backends (Docker, SSH, OpenShell).

```mermaid
flowchart TB
    subgraph Entry["ENTRY POINTS — 12 Options"]
        CLI["CLI Shell<br/>manusclaw"]
        SRV["HTTP/WS Server<br/>manusclaw-server"]
        MULTI["Multi-Agent<br/>manusclaw-multi"]
        CRON["Cron Scheduler<br/>manusclaw-cron"]
        SSH_E["SSH Gateway<br/>manusclaw-ssh"]
        VOICE["Voice Wake<br/>manusclaw voice"]
        WEBHOOK["Webhooks<br/>manusclaw-webhook"]
        GMAIL["Gmail Watch<br/>Pub/Sub"]
        CHAN["12+ Channels<br/>Telegram, Discord..."]
        FLOW["PlanningFlow<br/>run_flow.py"]
        MCP["MCP Client/Server<br/>run_mcp.py"]
        SESS["Session Tools<br/>manusclaw-sessions"]
    end

    subgraph Core["AGENT CORE — PAORR Loop"]
        PLAN["Plan<br/>Analyze task, create strategy"]
        ACT["Act<br/>Execute tools, generate code"]
        OBS["Observe<br/>Read results, capture output"]
        REFL["Reflect<br/>Evaluate progress vs goal"]
        RETRY["Retry<br/>Adapt and re-execute"]
        TOOLS["14+ Tools<br/>Bash, Python, Browser, Editor..."]
        MEM["3-Tier Memory<br/>Short + Session + Long"]
        SEC["Security Layer<br/>IdentityGuard + PermissionGate"]
    end

    subgraph LLM["LLM LAYER — Multi-Provider"]
        ROUTER["Universal Router<br/>10+ providers"]
        POOL["Credential Pool<br/>Auto-rotation + 60s cooldown"]
        FAIL["Cross-Provider Failover<br/>Priority-based fallback"]
        REDACT["Secret Redaction<br/>Keys hidden from logs"]
    end

    subgraph Integration["INTEGRATION LAYER"]
        CHAN2["12+ Messaging Adapters"]
        CANVAS["Live Canvas A2UI<br/>WebSocket real-time UI"]
        HOOKS["Webhooks + HMAC-SHA256"]
        NODES["Mobile/Desktop<br/>Canvas Node clients"]
        SAND["3 Sandbox Backends<br/>Docker / SSH / OpenShell"]
    end

    Entry --> Core
    PLAN --> ACT --> OBS --> REFL --> RETRY --> PLAN
    Core --> LLM
    Core --> Integration
    LLM --> ROUTER --> POOL --> FAIL
```

### The PAORR Reasoning Loop

The heart of ManusClaw is the **PAORR (Plan-Act-Observe-Reflect-Retry) loop** — a five-phase autonomous reasoning cycle that distinguishes it from simple request-response chatbots. Here is how each phase works:

1. **Plan**: The agent receives a task and analyzes it to determine the optimal strategy. It identifies which tools to use, estimates the number of steps required, and creates a structured execution plan. This phase leverages heuristic tool scoring to select the most relevant tools from the 14+ available options, avoiding unnecessary exploration.

2. **Act**: The agent executes the plan by invoking tools — running shell commands, writing Python code, browsing web pages, editing files, or sending messages. Each tool invocation is tracked with full input/output recording in the SessionDB.

3. **Observe**: After each action, the agent captures and analyzes the output. It reads terminal output, file contents, web page content, or API responses. The observation phase is critical — the agent does not blindly execute; it processes feedback at every step.

4. **Reflect**: The agent compares the current state against the original goal. It evaluates whether the task is complete, partially complete, or has encountered an error. This meta-cognitive step enables the agent to recognize when it is stuck, when a different approach is needed, or when the task is successfully finished.

5. **Retry**: If the task is not complete, the agent adapts its strategy based on reflections and loops back to the Plan phase. The retry mechanism includes error classification (transient vs. permanent), backoff strategies, and tool-switching when a particular approach consistently fails. The maximum number of steps is configurable (default: 30).

This loop continues autonomously until the goal is achieved or the step limit is reached. The agent maintains a sliding window context of the most recent interactions and can compress older context to stay within token budgets.

### Design Philosophy

ManusClaw follows several core design principles that set it apart from other agent frameworks:

- **Channel Agnosticism**: The agent logic is completely decoupled from communication channels. The same agent instance handles a task identically whether it arrives via Telegram, Discord, CLI, or webhook. This means features and fixes propagate across all channels simultaneously.

- **Progressive Complexity**: ManusClaw works immediately with zero configuration (MockLLM mode). As users add API keys, configure channels, and enable features, the framework progressively unlocks more capabilities. There is no "minimum viable configuration" barrier to entry.

- **Fail-Safe Defaults**: Every system is designed to degrade gracefully. If an LLM provider fails, the cross-provider rotator switches to the next. If a messaging channel lacks credentials, it enters stub mode. If Docker is unavailable, the sandbox falls back to SSH or OpenShell.

- **Production Security**: The Identity Guard scans every user message against 30+ jailbreak patterns. The Permission Gate classifies every tool invocation into three tiers (Allow/Ask/Deny). Secret redaction prevents API keys from leaking into logs. The SSH server uses restricted shells with command whitelisting.

- **Persistent Intelligence**: ManusClaw remembers across sessions. The SessionDB stores full conversation history with FTS5 search. The Long-Term memory persists insights across restarts. Session branching allows forking conversation trees for parallel exploration.

### v5.0.0 Feature Summary

| Category | Features | Status |
|---|---|---|
| **Intelligence** | PAORR loop, 14 tools, heuristic scoring, multi-agent orchestration | Stable |
| **LLM Providers** | 10+ providers: OpenAI, Anthropic, Groq, Gemini, Mistral, Bedrock, Ollama, GGUF, HuggingFace, OpenRouter + auto failover | Stable |
| **Messaging Channels** | 12+ adapters: Telegram, Discord, Slack, WhatsApp, Signal, Teams, Matrix, IRC, Twitch, WebChat, Email, Google Chat | Stable |
| **Voice** | Wake word (3 backends: Porcupine/Google/Stub), Talk Mode (mic->STT->agent->TTS), 3 TTS providers | Stable |
| **Canvas A2UI** | Live UI rendering via WebSocket — agents generate charts, tables, buttons in real-time | Stable |
| **Webhooks** | HMAC-SHA256 verified incoming webhooks with Jinja-style prompt templates | Stable |
| **SSH Gateway** | Full SSH server with restricted shell, public key auth, 9 whitelisted commands | Stable |
| **Gmail** | Google Cloud Pub/Sub push notifications, email processing, auto-reply | Stable |
| **Companion Apps** | macOS menubar, Windows system tray, Mobile Canvas Node | Stable |
| **Multi-Agent** | Role-based pipeline (PM -> Architect -> Engineer -> QA) + per-channel routing | Stable |
| **Sandboxes** | 3 backends: Docker, SSH, OpenShell (Linux namespace isolation) | Stable |
| **Session Tools** | CLI for session management: list, history, send, spawn, branch, export | Stable |
| **Model Failover** | Per-provider credential pools with auto-rotation, 60s cooldown, cross-provider fallback | Stable |
| **Enhanced Cron** | YAML-persisted jobs, multi-platform output routing, webhook delivery | Stable |

---

<br>

---

## 2 — Prerequisites

### System Requirements

| Requirement | Minimum | Recommended | Notes |
|---|---|---|---|
| **Python** | 3.11 | 3.12 or 3.13 | 3.14 compatible (uses `tomllib` from stdlib) |
| **RAM** | 512 MB | 2 GB | 4 GB+ required for local models via Ollama |
| **Disk Space** | 300 MB (core) | 2 GB (all features) | 4 GB+ with Playwright browser binaries |
| **Operating System** | Linux, macOS, Windows 10+, Android (Termux) | Ubuntu 22.04+, macOS 13+ | All major platforms supported |
| **Network** | Required for cloud LLMs | Stable broadband | Offline available with Ollama/GGUF backends |
| **Audio Hardware** | Not required | Microphone + speakers | Required for Voice Wake and Talk Mode |
| **Docker** | Not required | Docker 20.10+ | Required for Docker sandbox backend |
| **Node.js** | Not required | 18+ | Required for Playwright browser automation |

> **Important**: ManusClaw uses Python 3.11+ features including `tomllib` (3.11), `ExceptionGroup` (3.11), and `type` parameter in `TypeVar` (3.12). Python 3.10 and below are **not supported** and will fail on import.

### Platform Compatibility Matrix

| Platform | Supported | Installation Method | Notes |
|---|---|---|---|
| **Ubuntu/Debian** | Full | pip, install.sh, Docker | Primary development platform |
| **Fedora/RHEL** | Full | pip, Docker | May need `python3.11` from SCL |
| **macOS (Intel)** | Full | pip, install.sh, Docker, Homebrew | Tested on macOS 12+ |
| **macOS (Apple Silicon)** | Full | pip, install.sh, Docker | Native ARM support |
| **Windows 10/11** | Full | pip, install.ps1, Docker, WSL2 | PowerShell installer available |
| **WSL2 (Windows)** | Full | pip, install.sh | See [WSL2 Guide](docs/platforms/wsl.md) |
| **Android (Termux)** | Full | setup-termux.sh | See [Termux Guide](docs/platforms/termux.md) |
| **Google Colab** | Full | pip (per-session) | See [Colab Guide](docs/platforms/colab.md) |
| **iOS/iPadOS** | Limited | Remote only | Use via SSH or WebChat from a remote server |

### API Keys — What You Need

ManusClaw works **immediately without any API key** — it defaults to `MockLLM` mode, which returns placeholder responses suitable for testing the UI, tool dispatch, and session management. However, for real autonomous task execution, you need at least one LLM provider API key.

#### LLM Providers (pick one or more)

| Provider | Environment Variable | Free Tier | Speed | Recommended For |
|---|---|---|---|---|
| **Groq** | `GROQ_API_KEY` | Yes (rate-limited) | Fastest | Production use, fast iteration |
| **OpenAI** | `OPENAI_API_KEY` | No | Fast | Best model quality (GPT-4o, o1, o3) |
| **Anthropic** | `ANTHROPIC_API_KEY` | No | Medium | Claude 3.5/4 (excellent for code) |
| **Google Gemini** | `GOOGLE_API_KEY` | Yes (rate-limited) | Fast | Cost-effective, multimodal |
| **Mistral AI** | `MISTRAL_API_KEY` | Yes (limited) | Fast | European hosting, Mistral Large |
| **AWS Bedrock** | `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY` | No | Varies | Enterprise, data residency |
| **Ollama** | None required | Free (local) | N/A (local) | Privacy, offline, no API needed |
| **OpenRouter** | `OPENROUTER_API_KEY` | Varies | Varies | Unified access to 100+ models |

> **Recommendation**: For first-time users, get a free **Groq API key** from https://console.groq.com — it is the fastest free option and provides access to Llama 3.3 70B, Mixtral, and Gemma models.

#### Credential Pool (Multiple Keys)

ManusClaw supports up to **10 API keys per provider** for automatic rotation. When one key hits rate limits, it enters a 60-second cooldown and the next key is used automatically:

```bash
OPENAI_API_KEY=sk-primary-key
OPENAI_API_KEY_2=sk-secondary-key
OPENAI_API_KEY_3=sk-tertiary-key
# Up to OPENAI_API_KEY_9
```

This rotation happens transparently — the agent continues operating without interruption even when keys are exhausted.

#### Optional Feature API Keys

| Feature | Key(s) | Required? | Get It |
|---|---|---|---|
| **Voice Wake (Porcupine)** | `PICOVOICE_API_KEY` | No (falls back to Google STT) | https://picovoice.ai |
| **Voice TTS (ElevenLabs)** | `ELEVENLABS_API_KEY` | No (falls back to OpenAI/System) | https://elevenlabs.io |
| **Telegram** | `TELEGRAM_BOT_TOKEN` | Only for Telegram channel | @BotFather |
| **Discord** | `DISCORD_BOT_TOKEN` | Only for Discord channel | Discord Developer Portal |
| **Slack** | `SLACK_BOT_TOKEN` | Only for Slack channel | Slack App Directory |
| **WhatsApp** | `WHATSAPP_ACCESS_TOKEN` + `WHATSAPP_BUSINESS_PHONE_ID` | Only for WhatsApp | Meta Business API |
| **Signal** | `SIGNAL_CLI_REST_URL` + `SIGNAL_CLI_NUMBER` | Only for Signal | signal-cli |
| **Matrix** | `MATRIX_HOMESERVER` + `MATRIX_ACCESS_TOKEN` + `MATRIX_USER_ID` | Only for Matrix | matrix.org |
| **IRC** | `IRC_SERVER` + `IRC_PORT` + `IRC_NICK` + `IRC_CHANNELS` | Only for IRC | Any IRC server |
| **Twitch** | `TWITCH_BOT_TOKEN` + `TWITCH_CHANNEL` | Only for Twitch | Twitch Developer |
| **Microsoft Teams** | `MICROSOFT_APP_ID` + `MICROSOFT_APP_PASSWORD` | Only for Teams | Azure Bot Framework |
| **Google Chat** | `GOOGLE_CHAT_SERVICE_ACCOUNT` | Only for Google Chat | Google Cloud Console |
| **Email** | `EMAIL_SMTP_HOST` + `EMAIL_USER` + `EMAIL_PASS` | Only for Email | Any SMTP server |
| **SSH Server** | `MANUSCLAW_SSH_ENABLED=true` + auth keys | Only for SSH | Generate keys |
| **Gmail Automation** | `GOOGLE_APPLICATION_CREDENTIALS` | Only for Gmail | Google Cloud Console |
| **Server API Auth** | `MANUSCLAW_API_KEY` | Only for server auth | Generate your own |

> **Important**: Every messaging channel enters **stub mode** when its credentials are not configured. The server starts, logs a warning, and continues without that channel. This means you can configure channels incrementally — no channel blocks the startup of another.

---

<br>

---

## 3 — Installation Guide

### Quick Start — 5 Commands to Running

This is the fastest path from zero to a working ManusClaw agent:

```bash
# 1. Clone the repository
git clone https://github.com/ManusAgents/manusclaw.git && cd ManusClaw

# 2. Install the core framework
pip install -e .

# 3. Set an API key (Groq is free and fastest)
export GROQ_API_KEY="gsk_your-groq-key-here"

# 4. Launch ManusClaw
manusclaw

# That's it — you're running ManusClaw!
```

```mermaid
flowchart LR
    A["Clone<br/>git clone"] --> B["Install<br/>pip install -e ."]
    B --> C["API Key<br/>export GROQ_API_KEY"]
    C --> D["Launch<br/>manusclaw"]
    D --> E["Running!"]

    style A fill:#6c5ce7,stroke:#a29bfe,color:#fff
    style B fill:#00b894,stroke:#55efc4,color:#fff
    style C fill:#fdcb6e,stroke:#f9ca24,color:#2d3436
    style D fill:#e17055,stroke:#fab1a0,color:#fff
    style E fill:#00cec9,stroke:#81ecec,color:#2d3436
```

### Installation Methods

#### Method 1: pip (Recommended)

The pip method installs ManusClaw as an editable package, meaning code changes take effect immediately without reinstallation. This is the recommended method for development and production use.

```bash
# Clone
git clone https://github.com/ManusAgents/manusclaw.git
cd ManusClaw

# Core install (CLI + basic tools + MockLLM)
pip install -e .

# Full install (all v5.0 features)
pip install -e ".[all-plus]"
```

After installation, the following commands are available on your `PATH`:
- `manusclaw` — Interactive CLI shell
- `manusclaw-server` — HTTP/WS server
- `manusclaw-cron` — Cron scheduler
- `manusclaw-multi` — Multi-agent pipeline
- `manusclaw-sessions` — Session management
- `manusclaw-channels` — Channel management
- `manusclaw-webhook` — Webhook management

#### Method 2: Installer Scripts

ManusClaw includes platform-specific installer scripts that handle Python version checks, virtual environment creation, and dependency installation automatically.

**Linux / macOS**:
```bash
git clone https://github.com/ManusAgents/manusclaw.git
cd ManusClaw
chmod +x install.sh
./install.sh
```

**Windows (PowerShell)**:
```powershell
git clone https://github.com/ManusAgents/manusclaw.git
cd ManusClaw
.\install.ps1
```

**Android (Termux)**:
```bash
git clone https://github.com/ManusAgents/manusclaw.git
cd ManusClaw
chmod +x setup-termux.sh
./setup-termux.sh
```

#### Method 3: Docker

Docker deployment provides isolated, reproducible environments. The Dockerfile uses a multi-stage build for minimal image size.

```bash
# Clone (if not already)
git clone https://github.com/ManusAgents/manusclaw.git
cd ManusClaw

# Server mode — HTTP/WS server on port 8765
docker compose --profile server up -d

# Interactive CLI mode
docker compose run manusclaw

# Multi-Agent mode
docker compose --profile multi run manusclaw-multi "your goal here"
```

The Docker image:
- Uses `python:3.11-slim` as the base
- Creates a non-root `manusclaw` user for security
- Exposes port 8765
- Includes a healthcheck at `/healthz`
- Mounts persistent volumes for workspace, logs, and config
- Passes through `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, and `GOOGLE_API_KEY` from the environment

#### Method 4: Google Colab

Run ManusClaw in a free cloud environment with GPU access. See the [Colab Guide](docs/platforms/colab.md) for the complete notebook setup.

```bash
# In a Colab cell:
!git clone https://github.com/ManusAgents/manusclaw.git
%cd ManusClaw
!pip install -e ".[all]"
```

#### Method 5: WSL2 (Windows Subsystem for Linux)

Full native Linux experience on Windows with GPU passthrough. See the [WSL2 Guide](docs/platforms/wsl.md) for detailed setup.

### Optional Dependency Groups

ManusClaw uses Python's optional dependency system — install only what you need instead of pulling in every library:

| Group | Install Command | What You Get |
|---|---|---|
| **Core** | `pip install -e .` | CLI + basic tools + MockLLM |
| **All Features** | `pip install -e ".[all-plus]"` | Everything listed below |
| `server` | `pip install -e ".[server]"` | FastAPI, Uvicorn — HTTP/WS server |
| `cli` | `pip install -e ".[cli]"` | Rich, prompt_toolkit — interactive shell |
| `browser` | `pip install -e ".[browser]"` | Playwright, crawl4ai — web automation |
| `search` | `pip install -e ".[search]"` | DuckDuckGo search engine |
| `cron` | `pip install -e ".[cron]"` | croniter — scheduled tasks |
| `viz` | `pip install -e ".[viz]"` | matplotlib, mpld3 — data visualization |
| `voice` | `pip install -e ".[voice]"` | Porcupine, ElevenLabs, pyttsx3 — voice I/O |
| `ssh` | `pip install -e ".[ssh]"` | asyncssh — SSH server + sandbox |
| `gmail` | `pip install -e ".[gmail]"` | Google API client — email automation |
| `matrix` | `pip install -e ".[matrix]"` | matrix-nio — Matrix messaging |
| `companion` | `pip install -e ".[companion]"` | pystray, rumps — desktop tray apps |
| `mistral` | `pip install -e ".[mistral]"` | mistralai — Mistral AI provider |
| `bedrock` | `pip install -e ".[bedrock]"` | boto3 — AWS Bedrock provider |
| `google` | `pip install -e ".[google]"` | google-generativeai — Gemini provider |
| `ollama` | `pip install -e ".[ollama]"` | ollama — local model provider |

**Dependency Tree Visualization**:
```
manusclaw (core: pydantic, python-dotenv, pyyaml, openai, anthropic, aiohttp)
├── [server]      → fastapi, uvicorn[standard]
├── [cli]         → rich, prompt_toolkit
├── [cron]        → croniter
├── [browser]     → playwright, crawl4ai
├── [search]     → duckduckgo-search
├── [viz]        → matplotlib, mpld3
├── [voice]      → pvporcupine, sounddevice, speech-recognition, pyttsx3, elevenlabs
├── [ssh]        → asyncssh
├── [gmail]      → google-api-python-client, google-auth-httplib2
├── [matrix]     → matrix-nio
├── [companion]  → pystray, rumps
├── [mistral]    → mistralai
├── [bedrock]    → boto3
├── [google]     → google-generativeai
├── [ollama]     → ollama
├── [all]        → server + cli + cron + browser + search + viz + mistral + bedrock + google + ollama
└── [all-plus]   → all + voice + ssh + gmail + matrix + companion
```

### Verifying Installation

After installation, verify everything is working correctly:

```bash
# Check version
manusclaw --version

# Run tests (requires APP_ENV=test for MockLLM)
cd ManusClaw
APP_ENV=test pytest -n 4 -q

# Expected: 210 passed, 2 skipped in ~15 seconds
```

> **See also**: [Full Installation Guide](docs/installation.md) | [Platform-Specific Guides](docs/platforms/) | [Troubleshooting](docs/troubleshooting.md)

---

<br>

---

## 4 — Configuration Guide

### Configuration Priority Chain

ManusClaw uses a **seven-layer configuration cascade** — each layer overrides the previous one, and environment variables always win. This design allows you to set global defaults in `config.toml`, per-profile overrides in `config.yaml`, and runtime overrides via environment variables, all without conflicts.

```mermaid
flowchart TB
    E["1. Environment Variables<br/>GROQ_API_KEY, LLM_MODEL_OVERRIDE<br/>HIGHEST PRIORITY"]
    E --> PE["2. Profile .env<br/>~/.manusclaw/profiles/&lt;name&gt;/.env"]
    PE --> PC["3. Profile config.yaml<br/>~/.manusclaw/profiles/&lt;name&gt;/config.yaml"]
    PC --> GE["4. Global .env<br/>~/.manusclaw/.env"]
    GE --> GC["5. Global config.yaml<br/>~/.manusclaw/config.yaml"]
    GC --> CT["6. Local config.toml<br/>./config.toml (project root)"]
    CT --> D["7. Built-in Defaults<br/>MockLLM, safe defaults<br/>LOWEST PRIORITY"]

    style E fill:#e74c3c,stroke:#ff6b6b,color:#fff
    style PE fill:#e67e22,stroke:#f39c12,color:#fff
    style PC fill:#f1c40f,stroke:#f9ca24,color:#2d3436
    style GE fill:#27ae60,stroke:#2ecc71,color:#fff
    style GC fill:#2980b9,stroke:#3498db,color:#fff
    style CT fill:#8e44ad,stroke:#9b59b6,color:#fff
    style D fill:#636e72,stroke:#b2bec3,color:#fff
```

**Practical example**: If you set `provider = "ollama"` in `config.toml` but have `GROQ_API_KEY` in your environment, ManusClaw will auto-detect the Groq key and use it as the active provider, ignoring the TOML setting. This auto-detection scans for `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GOOGLE_API_KEY`, `MISTRAL_API_KEY`, `GROQ_API_KEY`, `OLLAMA_API_KEY`, and `AWS_ACCESS_KEY_ID` in that order.

### config.toml — Default Configuration

The `config.toml` file at the project root provides the legacy default configuration. For v5.0.0, prefer `config.yaml` which supports the richer structure including model profiles, agent routing, and channel flags.

```toml
[llm]
provider = "mock"                # mock | openai | azure | anthropic | ollama | openrouter | gguf | huggingface
model    = "gpt-4o"             # Default model name (provider-specific)
max_tokens   = 4096             # Max tokens per LLM response
temperature  = 0.0              # Temperature (0.0 = deterministic)
max_retries  = 6                 # Retry attempts on transient failures
timeout      = 1800             # 30-minute timeout for deep-reasoning models (o1, DeepSeek R1)

[browser]
headless          = true         # Run browser without visible window
disable_security  = false        # Disable same-origin policy (dangerous)
max_content_length = 10000       # Max characters to capture from web pages

[search]
engines     = ["duckduckgo", "bing"]   # Search engines to use
max_results = 10                         # Max results per search query

[sandbox]
enabled      = false             # Enable sandboxed code execution
docker_image = "python:3.11-slim"       # Docker image for sandbox
memory_limit = "256m"             # Memory limit for sandbox containers
timeout      = 30                 # Execution timeout in seconds

[runflow]
enable_data_analysis = false      # Enable data analysis agent in flow
timeout = 7200                   # 2-hour timeout for planning flows

[ssh]
enabled = false                   # Enable SSH server
port    = 2222                    # SSH listen port
host    = "0.0.0.0"              # Bind address

workspace_dir = "workspace"       # Working directory for agent operations
max_steps     = 30                 # Maximum PAORR loop iterations
```

### config.yaml — v5.0.0 Extended Configuration

The `config.yaml` file (located at `~/.manusclaw/config.yaml` or per-profile) supports v5.0.0 features that `config.toml` cannot express: model failover profiles, multi-agent routing definitions, voice configuration, SSH server settings, and per-channel enable flags.

```yaml
# ── Model Failover Profiles ──────────────────────────────────
# Each profile defines an ordered list of providers.
# When provider N fails or hits rate limits, provider N+1 is tried.
model_profiles:
  default:
    - provider: groq
      model: llama-3.3-70b-versatile
      priority: 1
      api_key: ${GROQ_API_KEY}          # Resolved from environment
    - provider: openai
      model: gpt-4o
      priority: 2
      api_key: ${OPENAI_API_KEY}
    - provider: anthropic
      model: claude-sonnet-4-20250514
      priority: 3
      api_key: ${ANTHROPIC_API_KEY}

  # Named profile for research tasks (can be selected at runtime)
  research:
    - provider: google
      model: gemini-2.5-pro
      priority: 1
    - provider: groq
      model: llama-3.3-70b-versatile
      priority: 2

  coding:
    - provider: anthropic
      model: claude-sonnet-4-20250514
      priority: 1
    - provider: openai
      model: gpt-4o
      priority: 2

# ── Agent Definitions & Routing ──────────────────────────────
agents:
  defaults:
    sandbox_mode: false                    # Default sandbox mode for all agents
  definitions:
    - name: manus
      class_path: app.agent.manus.Manus
      system_prompt: "You are ManusClaw, an autonomous AI agent..."
    - name: data_analyst
      class_path: app.agent.data_analysis.DataAnalysisAgent
      system_prompt: "You are a data analysis specialist..."
    - name: browser_agent
      class_path: app.agent.browser.BrowserAgent
      system_prompt: "You are a web browsing specialist..."
  routes:
    - pattern: "channel:telegram"           # All Telegram users
      agent: manus
      priority: 1
    - pattern: "channel:discord,#analytics"  # Discord #analytics channel
      agent: data_analyst
      priority: 2
    - pattern: "user_id:admin_user"         # Specific user across all channels
      agent: manus
      priority: 3
    - pattern: "channel:discord"           # All other Discord
      agent: browser_agent
      priority: 4

# ── Voice Configuration ─────────────────────────────────────
voice:
  wake_word: "hey manus"           # Custom wake phrase
  sensitivity: 0.5                # Detection sensitivity (0.0-1.0)
  tts_provider: auto               # auto | elevenlabs | openai | system | null
  stt_engine: auto                 # auto | google | whisper | stub

# ── SSH Server ─────────────────────────────────────────────
ssh_server:
  enabled: false                  # Must be explicitly enabled
  port: 2222
  host: "0.0.0.0"
  host_key: ~/.manusclaw/ssh/host_key
  auth_keys: ~/.manusclaw/ssh/authorized_keys

# ── Channel Enable Flags ──────────────────────────────────
channels:
  telegram: true
  discord: true
  slack: false
  whatsapp: false
  signal: false
  matrix: false
  irc: false
  twitch: false
  teams: false
  email: false
  google_chat: false
  webchat: true                    # Built-in, always available
```

### .env Complete Reference

Environment variables override all other configuration sources. Place them in `~/.manusclaw/.env` (global) or `~/.manusclaw/profiles/<name>/.env` (per-profile). A `.env.example` file is provided in the ManusClaw repository root.

```bash
# ══════════════════════════════════════════════════════════════
# LLM PROVIDERS
# ══════════════════════════════════════════════════════════════
GROQ_API_KEY=gsk_your-key                  # Groq (fastest — recommended)
OPENAI_API_KEY=sk-your-key                 # OpenAI (GPT-4o, o1, o3)
OPENAI_API_KEY_2=sk-backup-key             # Backup key for rotation
OPENAI_API_KEY_3=sk-tertiary-key           # Third key for rotation
# Up to OPENAI_API_KEY_9
ANTHROPIC_API_KEY=sk-ant-your-key           # Anthropic (Claude 3.5/4)
ANTHROPIC_API_KEY_2=sk-ant-backup           # Backup Anthropic key
GOOGLE_API_KEY=your-key                    # Google Gemini
MISTRAL_API_KEY=your-key                    # Mistral AI
AWS_ACCESS_KEY_ID=your-key                  # AWS Bedrock
AWS_SECRET_ACCESS_KEY=your-secret           # AWS Bedrock
AWS_DEFAULT_REGION=us-east-1                 # AWS region (default)
OLLAMA_API_KEY=your-key                     # Ollama Cloud API key

# Custom LLM endpoint (OpenAI-compatible, e.g., Together AI, Fireworks)
LLM_BASE_URL=https://api.groq.com/openai/v1
LLM_API_KEY=your-key
LLM_MODEL_OVERRIDE=llama-3.3-70b-versatile

# ══════════════════════════════════════════════════════════════
# SERVER
# ══════════════════════════════════════════════════════════════
MANUSCLAW_API_KEY=your-secret-api-key       # Bearer token for REST auth
MANUSCLAW_ALLOWED_ORIGINS=http://localhost,https://yourdomain.com  # CORS

# ══════════════════════════════════════════════════════════════
# MESSAGING CHANNELS
# ══════════════════════════════════════════════════════════════
TELEGRAM_BOT_TOKEN=your-bot-token            # Telegram (@BotFather)
DISCORD_BOT_TOKEN=your-bot-token             # Discord Developer Portal
SLACK_BOT_TOKEN=xoxb-your-token              # Slack App

WHATSAPP_ACCESS_TOKEN=EAA...                  # Meta Business API
WHATSAPP_BUSINESS_PHONE_ID=1234567890         # Phone number ID
WHATSAPP_WEBHOOK_VERIFY_TOKEN=manusclaw_verify # Verification token

SIGNAL_CLI_REST_URL=http://localhost:8080       # signal-cli REST API
SIGNAL_CLI_NUMBER=+1234567890                 # Signal phone number

MATRIX_HOMESERVER=https://matrix.org           # Matrix server
MATRIX_ACCESS_TOKEN=your-token                 # Matrix access token
MATRIX_USER_ID=@bot:matrix.org                 # Matrix user ID

IRC_SERVER=irc.libera.chat                      # IRC server
IRC_PORT=6697                                  # IRC port (TLS)
IRC_NICK=manusclaw-bot                         # IRC nickname
IRC_CHANNELS=#manusclaw                        # Comma-separated channels
IRC_PASS=                                      # IRC NickServ password

TWITCH_BOT_TOKEN=oauth:your-token              # Twitch OAuth
TWITCH_CHANNEL=your-channel                    # Twitch channel name
TWITCH_BOT_NICK=manusclaw-bot                  # Twitch bot name

MICROSOFT_APP_ID=your-app-id                   # Azure Bot Framework
MICROSOFT_APP_PASSWORD=your-password           # Azure Bot Framework

GOOGLE_CHAT_SERVICE_ACCOUNT=/path/to/sa.json   # Google Cloud SA
GOOGLE_CHAT_SERVICE_ACCOUNT_JSON='{"type":"service_account",...}'

EMAIL_SMTP_HOST=smtp.gmail.com                 # SMTP server
EMAIL_SMTP_PORT=587                             # SMTP port (STARTTLS)
EMAIL_IMAP_HOST=imap.gmail.com                  # IMAP server (for receiving)
EMAIL_IMAP_PORT=993                             # IMAP port (TLS)
EMAIL_USER=your@gmail.com                       # Email username
EMAIL_PASS=your-app-password                    # Email password/app password

# ══════════════════════════════════════════════════════════════
# VOICE
# ══════════════════════════════════════════════════════════════
PICOVOICE_API_KEY=your-key                      # Porcupine wake word detection
ELEVENLABS_API_KEY=your-key                     # ElevenLabs TTS (optional)

# ══════════════════════════════════════════════════════════════
# SSH SERVER
# ══════════════════════════════════════════════════════════════
MANUSCLAW_SSH_ENABLED=false                     # Must be explicitly true
MANUSCLAW_SSH_PORT=2222                          # SSH listen port
MANUSCLAW_SSH_HOST=0.0.0.0                       # Bind address
MANUSCLAW_SSH_HOST_KEY=~/.manusclaw/ssh/host_key  # Host key path
MANUSCLAW_SSH_AUTH_KEYS=~/.manusclaw/ssh/authorized_keys  # Auth keys

# ══════════════════════════════════════════════════════════════
# GMAIL AUTOMATION
# ══════════════════════════════════════════════════════════════
GOOGLE_APPLICATION_CREDENTIALS=/path/to/sa.json   # Service account
GOOGLE_CLIENT_ID=your-client-id                   # OAuth client ID
GOOGLE_CLIENT_SECRET=your-client-secret           # OAuth client secret
GOOGLE_GMAIL_TOKEN_PATH=~/.manusclaw/gmail_token.json  # Stored token
GMAIL_USER_ADDRESS=your@gmail.com                 # Gmail address
GMAIL_AUTO_REPLY=false                            # Enable auto-reply
GMAIL_LABEL_FILTER=INBOX                          # Label to watch
GMAIL_WATCH_TOPIC_NAME=projects/manusclaw-gmail   # Pub/Sub topic
GMAIL_PROJECT_ID=manusclaw-gmail                  # GCP project ID

# ══════════════════════════════════════════════════════════════
# SANDBOX
# ══════════════════════════════════════════════════════════════
SANDBOX_BACKEND=docker                            # docker | ssh | openshell
SSH_SANDBOX_HOST=sandbox.example.com               # SSH sandbox host
SSH_SANDBOX_USER=manusclaw                         # SSH sandbox user
SSH_SANDBOX_KEY_PATH=~/.ssh/sandbox_key            # SSH sandbox key
SSH_SANDBOX_PORT=22                                # SSH sandbox port

# ══════════════════════════════════════════════════════════════
# CRON SCHEDULER
# ══════════════════════════════════════════════════════════════
MANUSCLAW_CRON_FILE=~/.manusclaw/cron_jobs.yaml    # Jobs file path

# ══════════════════════════════════════════════════════════════
# GENERAL / MISC
# ══════════════════════════════════════════════════════════════
APP_ENV=dev                                       # dev | prod | test
MANUSCLAW_HOME=~/.manusclaw                       # Config/data home directory
MANUSCLAW_PROFILE=                                # Named profile for isolation
MANUSCLAW_REDACT=false                             # Redact API keys from logs
MANUSCLAW_WORKSPACE=workspace                      # Workspace directory
MANUSCLAW_SKILLS_DIR=workspace/.skills            # Custom skills directory
FAL_KEY=                                          # FAL.ai image generation key
```

### Configuration Profiles

Profiles allow you to maintain separate configurations for different environments (development, production, testing) or different users. Each profile has its own `.env` and `config.yaml`.

```bash
# Create a production profile
mkdir -p ~/.manusclaw/profiles/production
echo "GROQ_API_KEY=gsk_prod-key" > ~/.manusclaw/profiles/production/.env
echo "provider: groq" > ~/.manusclaw/profiles/production/config.yaml

# Create a development profile
mkdir -p ~/.manusclaw/profiles/dev
echo "GROQ_API_KEY=gsk_dev-key" > ~/.manusclaw/profiles/dev/.env

# Use a specific profile
MANUSCLAW_PROFILE=production manusclaw

# Use another profile
MANUSCLAW_PROFILE=dev manusclaw --model llama-3.3-70b-versatile
```

The profile resolution order:
1. Check `MANUSCLAW_PROFILE` environment variable
2. If set, look for `~/.manusclaw/profiles/<name>/.env` and `config.yaml`
3. If not set, use global `~/.manusclaw/.env` and `config.yaml`

### Test Mode

Setting `APP_ENV=test` forces ManusClaw into test mode:
- Provider is forced to `mock` (no real API calls)
- `max_steps` is reduced to 5
- Workspace uses a temporary directory
- This is what the test suite uses — all 210 tests run without API keys

```bash
APP_ENV=test pytest
```

> **See also**: [Full Configuration Guide](docs/configuration.md) | [.env.example](https://github.com/ManusAgents/manusclaw/blob/main/.env.example)

---

<br>

---

## 5 — Running Modes

ManusClaw offers **12 distinct entry points**, each optimized for a different use case. This section covers every mode in detail, including commands, flags, and practical workflows.

### Entry Point Overview

```mermaid
flowchart TB
    subgraph Interactive["INTERACTIVE MODES"]
        CLI["manusclaw<br/>Interactive CLI Shell"]
        SSH["manusclaw-ssh<br/>SSH Remote Gateway"]
        VOICE_T["manusclaw voice talk<br/>Voice Conversation"]
    end

    subgraph Server["SERVER MODES"]
        HTTP["manusclaw-server<br/>HTTP/WS API Server"]
        VOICE_W["manusclaw voice wake<br/>Voice Wake Detection"]
    end

    subgraph Batch["BATCH / AUTOMATED MODES"]
        SINGLE["manusclaw 'task'<br/>Single-Shot Execution"]
        MULTI["manusclaw-multi 'goal'<br/>Multi-Agent Pipeline"]
        CRON["manusclaw-cron --run<br/>Scheduled Tasks"]
        GMAIL["Gmail Pub/Sub Watch<br/>Email Automation"]
    end

    subgraph Management["MANAGEMENT & INFRASTRUCTURE"]
        SESSIONS["manusclaw-sessions<br/>Session Management"]
        CHANNELS["manusclaw-channels<br/>Channel Management"]
        WEBHOOK["manusclaw-webhook<br/>Webhook Management"]
        MCP_S["run_mcp_server.py<br/>MCP Server"]
    end

    style Interactive fill:#6c5ce7,stroke:#a29bfe,color:#fff
    style Server fill:#00b894,stroke:#55efc4,color:#fff
    style Batch fill:#e17055,stroke:#fab1a0,color:#fff
    style Management fill:#fdcb6e,stroke:#f9ca24,color:#2d3436
```

### Mode 1: Interactive CLI Shell

The default and most feature-rich mode. Launch an interactive Rich-powered terminal interface with slash commands, session management, background tasks, and live tool execution display.

```bash
# Launch interactive shell
manusclaw

# With options
manusclaw --skin ares --profile production --model gpt-4o

# Single-shot mode (execute and exit)
manusclaw "Create a Python web scraper for Hacker News"
```

**CLI Flags**:

| Flag | Description | Default |
|---|---|---|
| `--skin {default,ares,mono,slate}` | UI color theme | `default` |
| `--model MODEL` | Override LLM model | From config |
| `--profile NAME` | Use named config profile | None |
| `--no-color` | Disable ANSI colors | Colors on |
| `--version` | Show version and exit | — |

**Interactive Slash Commands**:

| Command | Description |
|---|---|
| `/model [name]` | Show current model or switch to a different one |
| `/skills` | List all loaded skills with descriptions |
| `/tools` | Show available tools and their permission tier |
| `/memory` | View and manage agent's MEMORY.md persistent notes |
| `/compress` | Compress conversation context to save tokens |
| `/new` | Start a fresh session (clears context) |
| `/resume` | List recent sessions and resume one |
| `/branch [description]` | Fork current session into a parallel branch |
| `/tasks` | Show background task queue status |
| `/bg <task>` | Submit a task to run in the background |
| `/sessions list` | List all stored sessions |
| `/sessions history <id>` | View conversation history for a session |
| `/sessions send <id> -m "text"` | Inject a message into an existing session |
| `/sessions spawn -p "task"` | Create and run a new session |
| `/help` | Show all available commands |
| `/exit` | Exit ManusClaw (background tasks continue running) |

### Mode 2: HTTP/WS Server

Launch a production-ready FastAPI server with REST endpoints, WebSocket streaming, built-in WebChat UI, Live Canvas, and webhook management.

```bash
# Start server (default: 0.0.0.0:8765)
manusclaw-server

# Custom host and port
manusclaw-server --host 127.0.0.1 --port 3000

# Development with hot-reload
python run_server.py --reload
```

**Server Endpoints**:

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/healthz` | Health check (Docker uses this) |
| `POST` | `/run` | Execute agent task asynchronously |
| `POST` | `/run/sync` | Execute agent task synchronously |
| `GET` | `/sessions` | List all sessions |
| `GET` | `/sessions/{id}/messages` | Get session conversation history |
| `GET` | `/sessions/{id}/tool_calls` | Get session tool call history |
| `GET` | `/tools` | List all available tools |
| `POST` | `/multi-agent` | Run multi-agent pipeline |
| `WS` | `/ws/{session_id}` | Real-time agent streaming (text) |
| `WS` | `/ws/canvas/{session_id}` | Live Canvas A2UI streaming (UI) |
| `WS` | `/ws/chat/{session_id}` | WebChat WebSocket |
| `POST` | `/webhooks/create` | Register new webhook |
| `GET` | `/webhooks` | List all webhooks |
| `POST` | `/webhooks/{hook_id}` | Trigger webhook (HMAC verified) |
| `DELETE` | `/webhooks/{hook_id}` | Unregister webhook |
| `GET` | `/webhooks/sign/{hook_id}` | Generate HMAC test signature |
| `GET` | `/chat` | Built-in WebChat UI (HTML page) |
| `GET` | `/canvas` | Canvas viewer UI (HTML page) |

**Authentication**: Set `MANUSCLAW_API_KEY` to require a Bearer token on all REST endpoints. The server automatically applies CORS based on `MANUSCLAW_ALLOWED_ORIGINS`.

### Mode 3: Multi-Agent Pipeline

Launch a role-based multi-agent pipeline where tasks are decomposed and handled by specialized agents.

```bash
# Run multi-agent with a goal
manusclaw-multi "Build a REST API with tests" --mode build

# Or use the run script
python run_multi_agent.py "Design and implement a dashboard"
```

The pipeline runs: **Product Manager** (decomposes requirements) -> **Architect** (designs solution) -> **Engineer** (implements code) -> **QA** (reviews and tests). Each agent runs the full PAORR loop independently.

### Mode 4: Cron Scheduler

Schedule recurring tasks with YAML persistence, multi-platform output routing, and webhook delivery.

```bash
# Start the scheduler daemon
manusclaw-cron --run

# Add a new scheduled job
manusclaw-cron --add daily-report "Daily Report" "0 9 * * *" \
  "Summarize yesterday's activity and generate a report" \
  --output telegram:1234567890 \
  --output-channel discord \
  --output-target #reports

# List all jobs
manusclaw-cron --list

# Force-trigger a job immediately
manusclaw-cron --trigger daily-report

# Remove a job
manusclaw-cron --remove daily-report
```

Cron jobs are persisted as YAML in `~/.manusclaw/cron_jobs.yaml` (or path set by `MANUSCLAW_CRON_FILE`).

### Mode 5: Voice Wake Word Detection

Start a background listener that activates the agent when the wake word is detected.

```bash
# With Porcupine (best quality, hardware-accelerated)
export PICOVOICE_API_KEY=your-key
manusclaw voice wake --start --word "hey manus" --sensitivity 0.7

# Without Porcupine (falls back to Google STT — free)
manusclaw voice wake --start --word "hey manus"
```

**Wake Word Backend Priority**:
1. **Porcupine** (if `PICOVOICE_API_KEY` set) — hardware-accelerated, low latency, always listening
2. **Google STT** (fallback) — uses speech_recognition library, subprocess-based, moderate latency
3. **Stub** (testing) — simulates detection every 30 seconds for testing without audio hardware

### Mode 6: Voice Talk Mode

Enter a continuous voice conversation loop: wake word -> microphone -> STT -> agent processing -> TTS response -> loop.

```bash
# Start voice conversation
export GROQ_API_KEY=your-key
export ELEVENLABS_API_KEY=your-key        # Optional — better voice quality
manusclaw voice talk --start

# Or with OpenAI TTS (uses same API key as LLM)
export OPENAI_API_KEY=your-key
manusclaw voice talk --start

# Or with system TTS (free, offline, pyttsx3)
manusclaw voice talk --start
```

**TTS Provider Priority**:
1. **ElevenLabs** (if `ELEVENLABS_API_KEY` set) — premium quality, MP3 output
2. **OpenAI TTS** (if `OPENAI_API_KEY` set) — good quality, tts-1/tts-1-hd models
3. **System TTS** (pyttsx3) — offline, OS-native voices, WAV output
4. **NullTTS** — silent stub, logs text to console (for headless/testing)

**STT Engine Options**:
- `google` (default) — Google via speech_recognition, requires internet
- `whisper` — OpenAI Whisper, local processing, temp WAV files
- `stub` — stdin text input for testing without microphone

**Stop Phrases**: Say "stop listening", "go to sleep", "goodbye", "stop", or "end" to exit talk mode.

### Mode 7: SSH Remote Gateway

Start an SSH server that provides remote shell access to the ManusClaw agent with a restricted command set.

```bash
# Enable and start SSH server
export MANUSCLAW_SSH_ENABLED=true
export MANUSCLAW_SSH_PORT=2222
export MANUSCLAW_SSH_AUTH_KEYS=~/.manusclaw/ssh/authorized_keys
manusclaw-ssh start

# Connect from another machine
ssh admin@your-server -p 2222
```

**Available SSH Commands** (restricted shell whitelist):
| Command | Description |
|---|---|
| `status` | Show ManusClaw status (running agents, uptime) |
| `restart` | Restart the ManusClaw agent |
| `logs` | View recent logs |
| `agent <prompt>` | Send a prompt to the agent |
| `channels list` | List active messaging channels |
| `cron list` | List cron jobs |
| `help` | Show available commands |
| `exit` | Disconnect from SSH |

### Mode 8: Messaging Channels

Start and manage individual messaging channel adapters. Each channel runs as a long-polling or webhook listener.

```bash
# Start specific channels
manusclaw-channels start telegram
manusclaw-channels start discord
manusclaw-channels start whatsapp

# List configured (enabled) channels
manusclaw-channels

# All channels can also be started via the server
manusclaw-server   # Starts all enabled channels
```

Each channel adapter enters **stub mode** when credentials are missing — it logs a warning and continues without blocking other channels.

### Mode 9: Session Management CLI

Manage agent sessions outside of the interactive shell — list history, inject messages, spawn new sessions, export data.

```bash
# List all sessions
manusclaw-sessions list

# View conversation history
manusclaw-sessions history <session-id> --tool-calls --limit 50

# Inject a message into an existing session
manusclaw-sessions send <session-id> -m "Continue the previous analysis"

# Create and run a new session
manusclaw-sessions spawn -p "Analyze the Q2 sales data" --agent manus --mode build

# Delete a session
manusclaw-sessions delete <session-id> --force

# Export session as JSON
manusclaw-sessions export <session-id> -o session.json --include-tool-calls
```

### Mode 10: Webhook Management

Register, list, and manage webhooks for integrating external events with the ManusClaw agent.

```bash
# Create a webhook (e.g., for GitHub push events)
manusclaw-webhook create \
  --id github-push \
  --url "/webhooks/github-push" \
  --secret "my-webhook-secret" \
  --prompt "Analyze this push event: {{payload.head_commit.message}} by {{payload.sender.login}}" \
  --session my-analysis-session

# List all webhooks
manusclaw-webhook --list

# Get HMAC signature for testing
manusclaw-webhook sign --id github-push --payload '{"test": "data"}'

# Remove a webhook
manusclaw-webhook --remove github-push
```

Webhook prompts support Jinja-style `{{payload.field}}` template interpolation for extracting data from incoming payloads. HMAC-SHA256 signatures are verified using the `X-Hub-Signature-256` header.

### Mode 11: PlanningFlow

Launch a specialized planning flow that decomposes complex goals into structured execution plans.

```bash
python run_flow.py "Build an e-commerce website with payment processing"
```

### Mode 12: MCP (Model Context Protocol)

Connect ManusClaw to external tools and services via the Model Context Protocol.

```bash
# Run as MCP client
python run_mcp.py --connection stdio --server-url "path/to/server" --interactive

# Run as MCP server
python run_mcp_server.py --host 0.0.0.0 --port 3001
```

> **See also**: [Full Usage Guide](docs/usage.md) | [Deployment Guide](docs/deployment.md)

---

<br>

---

## 6 — Feature Showcase

### 6.1 — 12+ Messaging Channels

ManusClaw communicates across 12+ messaging platforms through a unified adapter architecture. Each channel implements `BaseMessagingAdapter` with methods for `send()`, `receive()`, and `start()`. The **MessagingGateway** manages all adapters simultaneously, caches agent instances with LRU eviction (128 max, 5-minute idle TTL), and routes messages through the **AgentRouter** for per-channel agent selection.

| Channel | Auth Method | Connection Type | Required Env Vars |
|---|---|---|---|
| **Telegram** | Bot Token | HTTP Long Polling | `TELEGRAM_BOT_TOKEN` |
| **Discord** | Bot Token | REST API (v10) | `DISCORD_BOT_TOKEN` |
| **Slack** | Bot Token | REST API | `SLACK_BOT_TOKEN` |
| **WhatsApp** | Cloud API Token | Webhook (incoming) + REST (outgoing) | `WHATSAPP_ACCESS_TOKEN`, `WHATSAPP_BUSINESS_PHONE_ID` |
| **Signal** | REST URL + Phone | HTTP REST | `SIGNAL_CLI_REST_URL`, `SIGNAL_CLI_NUMBER` |
| **Microsoft Teams** | App ID + Password | Bot Framework | `MICROSOFT_APP_ID`, `MICROSOFT_APP_PASSWORD` |
| **Matrix** | Access Token | WebSocket (matrix-nio) | `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN`, `MATRIX_USER_ID` |
| **IRC** | Nick + Password | IRC Protocol | `IRC_SERVER`, `IRC_PORT`, `IRC_NICK`, `IRC_CHANNELS` |
| **Google Chat** | Service Account | Pub/Sub + REST | `GOOGLE_CHAT_SERVICE_ACCOUNT` |
| **Twitch** | OAuth Token | IRC Protocol | `TWITCH_BOT_TOKEN`, `TWITCH_CHANNEL`, `TWITCH_BOT_NICK` |
| **WebChat** | None (built-in) | WebSocket | None |
| **Email** | SMTP/IMAP Creds | SMTP (outgoing) + IMAP (incoming) | `EMAIL_SMTP_HOST`, `EMAIL_USER`, `EMAIL_PASS` |

**Key behaviors**:
- Unconfigured channels enter **stub mode** — they log a warning but do not block other channels
- The gateway maintains a per-user agent cache (128 instances max, LRU eviction with 5-minute idle TTL)
- When AgentRouter is configured, different channels can route to different specialized agents
- All channels share the same underlying PAORR loop, tools, memory, and LLM providers

### 6.2 — Voice Wake Word & Talk Mode

Voice features enable hands-free interaction with ManusClaw through a continuous listen-process-speak cycle.

**Wake Word Detection**:
```bash
# Best quality (Porcupine — hardware accelerated)
export PICOVOICE_API_KEY=your-key
manusclaw voice wake --start --word "hey manus" --sensitivity 0.7

# Free fallback (Google STT — no API key needed)
manusclaw voice wake --start --word "hey manus"
```

**Talk Mode** (continuous voice conversation):
```bash
# Full conversation loop with ElevenLabs TTS
export GROQ_API_KEY=your-key
export ELEVENLABS_API_KEY=your-key
manusclaw voice talk --start
```

The talk mode flow: **wake word detected** -> **listen (microphone)** -> **STT (speech-to-text)** -> **agent.run(prompt)** -> **PAORR loop** -> **response text** -> **TTS (text-to-speech)** -> **speak (speakers)** -> **loop back to listen**.

```mermaid
flowchart LR
    W["Wake Word<br/>'hey manus'"] --> L["Listen<br/>Microphone"]
    L --> STT["STT<br/>Google/Whisper"]
    STT --> AG["Agent<br/>PAORR Loop"]
    AG --> TTS["TTS<br/>ElevenLabs/OpenAI/System"]
    TTS --> SP["Speak<br/>Speakers"]
    SP --> |"Continue?"| L

    style W fill:#e74c3c,color:#fff
    style L fill:#e67e22,color:#fff
    style STT fill:#f1c40f,color:#2d3436
    style AG fill:#6c5ce7,color:#fff
    style TTS fill:#00b894,color:#fff
    style SP fill:#2980b9,color:#fff
```

### 6.3 — Live Canvas (A2UI Protocol)

The **Agent-to-UI (A2UI) protocol** enables agents to render interactive content in real-time through WebSocket connections. Instead of just sending text, the agent can push charts, tables, progress bars, and interactive buttons to connected clients.

```bash
# Start the server with Canvas support
manusclaw-server
# Canvas is available at: ws://localhost:8765/ws/canvas/{session_id}
# Canvas viewer at: http://localhost:8765/canvas
```

**A2UI Message Types**:
- `sync` — Full UI state replacement (initial render)
- `update` — Partial UI update (patch existing state)
- `clear` — Clear the canvas
- `event` — User interaction event (button click, form submit)

The Canvas tool is available to the agent as part of the 14+ tool set. When the agent decides to visualize data or present interactive elements, it uses the Canvas tool which sends A2UI messages over WebSocket.

### 6.4 — Webhooks (HMAC-SHA256 Verified)

ManusClaw can receive and process webhooks from external services (GitHub, Stripe, CI/CD pipelines, monitoring systems) with cryptographic signature verification.

```bash
# Register a webhook for GitHub push events
manusclaw-webhook create \
  --id github-push \
  --url "/webhooks/github-push" \
  --secret "whsec_your-secret-here" \
  --prompt "Analyze this push: {{payload.head_commit.message}} by {{payload.sender.login}}. Summary: {{payload.commits.length}} commits."

# Test the signature
manusclaw-webhook sign --id github-push --payload '{"head_commit": {"message": "fix: resolve null pointer"}}'

# The webhook is now available at POST /webhooks/github-push
# The sender must include header: X-Hub-Signature-256: sha256=<hex_digest>
```

Webhook prompt templates support `{{payload.field.nested.field}}` interpolation for extracting any data from the incoming JSON payload. The HMAC-SHA256 signature is verified before the prompt is generated, ensuring authenticity.

### 6.5 — SSH Remote Gateway

The SSH server provides remote shell access to ManusClaw through a restricted command set with public key authentication.

```bash
# 1. Generate SSH host key (auto-generated on first run)
# 2. Set up authorized keys
mkdir -p ~/.manusclaw/ssh
cp ~/.ssh/id_rsa.pub ~/.manusclaw/ssh/authorized_keys

# 3. Enable and start
export MANUSCLAW_SSH_ENABLED=true
export MANUSCLAW_SSH_PORT=2222
export MANUSCLAW_SSH_AUTH_KEYS=~/.manusclaw/ssh/authorized_keys
manusclaw-ssh start

# 4. Connect
ssh admin@your-server -p 2222
```

The SSH server uses `asyncssh` for the connection layer and routes commands through a **RestrictedShell** with 9 whitelisted commands (`status`, `restart`, `logs`, `agent`, `channels list`, `cron list`, `help`, `exit`). All other commands are rejected. The server auto-generates an RSA 2048 host key if one does not exist.

### 6.6 — Gmail Automation

ManusClaw can watch your Gmail inbox via Google Cloud Pub/Sub and automatically process, categorize, or reply to emails.

```bash
# Set up Google Cloud service account credentials
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
export GMAIL_WATCH_TOPIC_NAME=projects/manusclaw-project/topics/gmail-push
export GMAIL_USER_ADDRESS=your@gmail.com

# Enable auto-reply (optional)
export GMAIL_AUTO_REPLY=true

# Start watching (runs alongside cron scheduler)
manusclaw-cron --run --gmail-watch
```

The Gmail integration uses:
- **Google Cloud Pub/Sub** for push notifications (instant, no polling)
- **Service account credentials** for server-to-server auth
- **Email processing** via the agent's PAORR loop (categorize, summarize, reply)
- **Auto-reply** mode for automated responses based on agent decisions

### 6.7 — Multi-Agent Routing

The **AgentRouter** enables per-channel and per-user agent selection, allowing you to run specialized agents for different contexts.

```yaml
# config.yaml
agents:
  definitions:
    - name: manus
      class_path: app.agent.manus.Manus
      system_prompt: "You are ManusClaw, a general-purpose AI agent..."
    - name: coder
      class_path: app.agent.manus.Manus
      system_prompt: "You are a coding specialist. Only write code."
    - name: analyst
      class_path: app.agent.data_analysis.DataAnalysisAgent
  routes:
    - pattern: "channel:telegram"
      agent: manus
      priority: 1
    - pattern: "channel:discord,#coding"
      agent: coder
      priority: 2
    - pattern: "user_id:admin_*"
      agent: manus
      priority: 3
```

**Route Pattern Types**:
- `channel:<platform>` — Route all messages from a channel to a specific agent
- `channel:<platform>,<target>` — Route messages from a specific channel target (e.g., Discord channel)
- `user_id:<regex>` — Route messages from users matching a regex pattern

Routes are evaluated in priority order (lower number = higher priority). The first matching rule wins.

### 6.8 — Model Failover & Credential Rotation

ManusClaw automatically handles API key exhaustion and provider failures without interrupting the agent's workflow.

**Credential Pool** (per-provider key rotation):
```bash
# Configure multiple keys for automatic rotation
OPENAI_API_KEY=sk-primary
OPENAI_API_KEY_2=sk-backup-1
OPENAI_API_KEY_3=sk-backup-2
```

When key 1 hits rate limits, it enters a 60-second cooldown and key 2 is used. When key 2 is also exhausted, key 3 is tried. If all keys are exhausted, the provider enters a full cooldown.

**Cross-Provider Failover** (via model profiles):
```yaml
model_profiles:
  default:
    - provider: groq, model: llama-3.3-70b-versatile, priority: 1
    - provider: openai, model: gpt-4o, priority: 2
    - provider: anthropic, model: claude-sonnet-4-20250514, priority: 3
```

If Groq fails completely (all keys exhausted), ManusClaw automatically switches to OpenAI, then Anthropic, without any user intervention.

**Retry Logic**: Up to 8 retries with exponential backoff + jitter (1s to 60s max). Transient errors (connection issues, HTTP 5xx) trigger retries. Non-transient errors (authentication, invalid requests) do not.

**Adaptive Timeouts**: Deep-thinking models (o1, DeepSeek R1, Claude 3.7 Sonnet) automatically get 30-minute timeouts. Regular models get the configured timeout (default: 1800s).

### 6.9 — Session Management

ManusClaw persists full conversation history with session branching, cross-session search, and CLI management tools.

**SessionDB** (SQLite WAL + FTS5):
- Stores sessions, messages, and tool calls in SQLite with WAL mode for concurrent access
- FTS5 virtual tables enable full-text search across all conversations
- Session branching (fork) allows parallel exploration of different approaches
- Context compression reduces token usage while preserving key information

```bash
# CLI session management
manusclaw-sessions list                            # List all sessions
manusclaw-sessions history <id> --tool-calls       # View with tool calls
manusclaw-sessions send <id> -m "continue"         # Inject message
manusclaw-sessions spawn -p "new task"             # Create new session
manusclaw-sessions export <id> -o data.json        # Export as JSON
```

### 6.10 — Enhanced Cron Scheduler

The cron scheduler persists jobs as YAML, supports multi-platform output routing, and can deliver results via webhooks.

```bash
# Add a job with multi-platform output
manusclaw-cron --add morning-brief "Morning Briefing" "0 8 * * 1-5" \
  "Generate a summary of overnight activity and today's priorities" \
  --output telegram:1234567890 \
  --output discord:#general

# Add a job with webhook delivery
manusclaw-cron --add ci-alert "CI Alert Handler" "*/5 * * * *" \
  "Monitor and respond to CI/CD failures" \
  --trigger-webhook https://your-ci.com/webhook

# Start the scheduler
manusclaw-cron --run
```

### 6.11 — Three Sandbox Backends

Execute untrusted code in isolated environments:

| Backend | Config | Requirements | Best For |
|---|---|---|---|
| **Docker** | `SANDBOX_BACKEND=docker` | Docker 20.10+ | General-purpose, most secure |
| **SSH** | `SANDBOX_BACKEND=ssh` | Remote server with SSH access | Remote execution, cloud VMs |
| **OpenShell** | `SANDBOX_BACKEND=openshell` | Linux kernel | Lightweight, no Docker needed |

```bash
# Docker sandbox (default)
export SANDBOX_BACKEND=docker
export DOCKER_IMAGE=python:3.12-slim
export SANDBOX_MEMORY_LIMIT=2g
export SANDBOX_TIMEOUT=300

# SSH sandbox (remote server)
export SANDBOX_BACKEND=ssh
export SSH_SANDBOX_HOST=sandbox.example.com
export SSH_SANDBOX_USER=manusclaw
export SSH_SANDBOX_KEY_PATH=~/.ssh/sandbox_key
```

### 6.12 — Companion Apps

Desktop and mobile applications that provide native access to ManusClaw:

| Platform | App | Type | Requirements |
|---|---|---|---|
| **macOS** | Menu Bar | System tray (pystray/rumps) | `[companion]` extras |
| **Windows** | Hub | System tray | `[companion]` extras |
| **Mobile** | Node Client | Canvas Node protocol | WebSocket connection to server |

### 6.13 — 10+ LLM Providers

ManusClaw supports a universal LLM router that connects to any OpenAI-compatible API:

| Provider | Models | Auth | Notes |
|---|---|---|---|
| **OpenAI** | GPT-4o, o1, o3, GPT-4o-mini | `OPENAI_API_KEY` | Best quality for code and reasoning |
| **Anthropic** | Claude 3.5 Sonnet, Claude 4 | `ANTHROPIC_API_KEY` | Excellent for long context and code |
| **Groq** | Llama 3.3 70B, Mixtral, Gemma 2 | `GROQ_API_KEY` | Fastest inference, free tier |
| **Google Gemini** | Gemini 2.5 Pro, Gemini Flash | `GOOGLE_API_KEY` | Multimodal, free tier |
| **Mistral AI** | Mistral Large, Codestral, Medium | `MISTRAL_API_KEY` | European hosting, fast |
| **AWS Bedrock** | Claude, Titan, Llama (via AWS) | `AWS_ACCESS_KEY_ID` | Enterprise, data residency |
| **Ollama** | Any GGUF model (local) | None | Privacy, offline, free |
| **OpenRouter** | 100+ models via unified API | `OPENROUTER_API_KEY` | Access to rare models |
| **GGUF** | Local quantized models | None | CPU inference, no server |
| **HuggingFace** | Any HF model with inference API | `HF_TOKEN` | Open-source models |
| **Custom** | Any OpenAI-compatible endpoint | `LLM_BASE_URL` + `LLM_API_KEY` | Self-hosted, Together AI, Fireworks |

### 6.14 — 14+ Built-in Tools

The agent has access to these tools, selected via heuristic scoring during the Plan phase:

| Tool | Permission | Description |
|---|---|---|
| `bash` | ASK | Execute shell commands |
| `python_execute` | ASK | Run Python code in sandbox |
| `web_search` | ALLOW | Search the web (DuckDuckGo) |
| `crawl4ai` | ALLOW | Crawl and extract web page content |
| `browser_use_tool` | ASK | Full browser automation (Playwright) |
| `str_replace_editor` | ASK | Read/write/edit files |
| `image_gen` | ALLOW | Generate images (FAL.ai) |
| `ask_human` | ALLOW | Request human input |
| `memory_tool` | ALLOW | Read/write persistent memory |
| `planning` | ALLOW | Create/modify execution plans |
| `data_viz` | ALLOW | Generate matplotlib charts |
| `delegate` | ASK | Delegate subtasks to sub-agents |
| `terminate` | ALLOW | Signal task completion |
| `canvas` | ALLOW | Render UI via A2UI protocol |
| `node_execute` | ASK | Execute on Canvas Node devices |
| `platform_control` | ASK | Control system platforms |
| `skill_manager` | ALLOW | Load/manage skill sets |
| `cross_session_search` | ALLOW | Search across all sessions |
| `selector` | ALLOW | Select between options |

> **See also**: [Full Feature Showcase](docs/features.md) | [Architecture](docs/ARCHITECTURE.md)

---

<br>

---

## 7 — Security & Memory

### 7.1 — Identity Guard (Anti-Jailbreak)

The **IdentityGuard** is a proactive defense system that scans every user message against **30+ regex patterns** before the agent processes it. It protects against:

| Attack Type | Example Patterns Detected |
|---|---|
| **Direct Override** | "ignore previous instructions", "disregard all rules", "forget everything above" |
| **Identity Manipulation** | "you are now X", "pretend you are", "act as DAN" |
| **System Prompt Extraction** | "reveal your system prompt", "repeat your instructions", "show me your rules" |
| **Token Injection** | `<|im_start|>`, `[system]:`, `<<SYS>>`, `<|endoftext|>` |
| **DAN-Style Attacks** | "do anything now", "DAN mode", "bypass restrictions", "no limits" |
| **Role Confusion** | "you are the admin", "I am your developer", "emergency override" |

When a pattern is detected:
1. The malicious content is **sanitized** from the message
2. The agent's identity is **reinforced** via a system message
3. The sanitized message continues to be processed normally
4. The attempt is **logged** for security auditing

### 7.2 — Permission Gate (Tool Authorization)

Every tool invocation passes through a three-tier **Permission Gate** that classifies the operation and determines whether it can proceed:

| Tier | Color | Tools | Behavior |
|---|---|---|---|
| **ALLOW** | Green | `MemoryTool`, `AskHuman`, `Terminate`, `web_search`, `crawl4ai`, `planning`, `data_viz`, `skill_manager`, `cross_session_search`, `selector`, `canvas` | Auto-approved, no confirmation needed |
| **ASK** | Yellow | `Bash`, `PythonExecute`, `BrowserUse`, `StrReplaceEditor`, `delegate`, `node_execute`, `platform_control` | Requires approval in Plan mode; auto-approved in Build mode |
| **DENY** | Red | Catastrophic operations (rm -rf /, fork bombs, privilege escalation) | Always blocked, regardless of mode |

The gate also checks against patterns like `rm -rf /`, `:(){ :|:& };:`, `dd if=/dev/zero`, and other destructive commands — these are hard-blocked even if the agent is in Build mode.

### 7.3 — Credential Pool & Rotation

API keys are never exposed in a single point of failure. The **CredentialPool** manages multiple keys per provider with automatic rotation:

```mermaid
flowchart LR
    REQ["API Request"] --> POOL["Credential Pool<br/>Key 1 (active)"]
    POOL --> |"Success"| LLM["LLM Provider"]
    POOL --> |"Rate Limited"| COOL["Key 1 → 60s Cooldown"]
    COOL --> POOL2["Key 2 (active)"]
    POOL2 --> |"Rate Limited"| COOL2["Key 2 → 60s Cooldown"]
    COOL2 --> POOL3["Key 3 (active)"]
    POOL3 --> |"All Exhausted"| XPROV["Cross-Provider<br/>Failover"]
    XPROV --> NEXT["Next Provider<br/>(from model_profiles)"]

    style REQ fill:#6c5ce7,color:#fff
    style POOL fill:#00b894,color:#fff
    style COOL fill:#fdcb6e,color:#2d3436
    style XPROV fill:#e74c3c,color:#fff
    style NEXT fill:#2980b9,color:#fff
```

**Features**:
- Up to 10 keys per provider (`OPENAI_API_KEY` through `OPENAI_API_KEY_9`)
- Automatic 60-second cooldown on rate-limited keys
- Priority-ordered key selection
- Cross-provider failover via `model_profiles`
- Connection-level retry with exponential backoff + jitter (1s to 60s)
- Transient vs non-transient error classification

### 7.4 — Secret Redaction

When `MANUSCLAW_REDACT=true` (or `LOG_REDACT_SECRETS=true`), all API keys are redacted from log output before being written. The `SecretRedactor` scans log messages for patterns matching API key formats (sk-..., gsk_..., xoxb-..., etc.) and replaces them with `[REDACTED]`. This prevents accidental key leakage in log files, CI/CD pipelines, and monitoring dashboards.

```bash
# Enable secret redaction
export MANUSCLAW_REDACT=true
manusclaw-server

# Logs will show: "Using API key: [REDACTED]" instead of the actual key
```

### 7.5 — Three-Tier Memory System

ManusClaw implements a three-tier memory architecture that provides both short-term working memory and long-term persistent intelligence:

```mermaid
flowchart TB
    subgraph Short["SHORT-TERM MEMORY"]
        SW["Sliding Window<br/>Last 100 messages<br/>Session-scoped"]
    end

    subgraph Session["SESSION MEMORY (SessionDB)"]
        DB["SQLite WAL + FTS5<br/>Full conversation history<br/>Tool call records<br/>Persists across restarts"]
        BR["Session Branching<br/>Fork conversations<br/>Parallel exploration"]
        CP["Context Compression<br/>Reduce token usage<br/>Preserve key info"]
        FTS["Full-Text Search<br/>Search all messages<br/>Cross-session queries"]
    end

    subgraph Long["LONG-TERM MEMORY"]
        LT["Persistent Storage<br/>SQLite + FTS5<br/>Cross-session insights<br/>MEMORY.md file"]
    end

    Short --> Session
    Session --> Long

    style Short fill:#6c5ce7,stroke:#a29bfe,color:#fff
    style Session fill:#00b894,stroke:#55efc4,color:#fff
    style Long fill:#e17055,stroke:#fab1a0,color:#fff
```

| Tier | Storage | Persistence | Size | Use Case |
|---|---|---|---|---|
| **Short-Term** | In-memory list | Session only | ~100 messages | Active conversation context, PAORR loop awareness |
| **SessionDB** | SQLite WAL + FTS5 | Survives restarts | Unlimited | Conversation history, tool call records, session branching, search |
| **Long-Term** | SQLite + FTS5 + MEMORY.md | Permanent | Unlimited | Cross-session insights, persistent knowledge, agent memory |

**Key capabilities**:
- **Session Branching**: Fork any session into parallel branches for exploring different approaches
- **Cross-Session Search**: FTS5-powered full-text search across all sessions and messages
- **Context Compression**: When token budgets are approached, older messages are compressed while preserving key information
- **MEMORY.md**: The agent maintains a persistent markdown file with long-term notes, preferences, and learned patterns

### 7.6 — SSH Security

The SSH server implements multiple security measures:

- **Public Key Authentication Only**: No password authentication. Users must have their public key in `~/.manusclaw/ssh/authorized_keys`
- **Restricted Shell**: Only 9 whitelisted commands are accepted (`status`, `restart`, `logs`, `agent`, `channels list`, `cron list`, `help`, `exit`)
- **Auto-Generated Host Key**: RSA 2048 host key generated on first run if missing
- **Connection Tracking**: Active connections are tracked for graceful shutdown
- **Explicit Opt-In**: SSH is disabled by default; must be explicitly enabled via `MANUSCLAW_SSH_ENABLED=true`

### 7.7 — Webhook Security

Webhook endpoints use **HMAC-SHA256** signature verification:

1. Each webhook is registered with a unique secret
2. Incoming requests must include `X-Hub-Signature-256: sha256=<hex_digest>` header
3. The server computes the HMAC of the request body using the webhook's secret
4. If signatures don't match, the request is rejected with HTTP 403
5. A `/webhooks/sign/{hook_id}` endpoint generates test signatures for debugging

### 7.8 — Test Mode Security

Setting `APP_ENV=test` provides a secure testing environment:
- Forces `provider="mock"` — no real API calls are made
- Reduces `max_steps` to 5 — limits execution depth
- Uses temporary workspace directories — no file system side effects
- Uses temporary databases — no session data pollution

> **See also**: [Full Troubleshooting Guide](docs/troubleshooting.md) | [Security Best Practices](docs/deployment.md)

---

<br>

---

## Testing

ManusClaw includes 210 unit tests covering every major subsystem. All tests use `APP_ENV=test` for MockLLM — no API keys are required.

```bash
# Run all tests (parallel, 4 workers — default)
pytest

# Verbose output
pytest -v

# Specific test suites
pytest tests/test_agent_loop.py tests/test_paorr.py
pytest tests/test_voice.py tests/test_canvas.py tests/test_ssh.py
pytest tests/test_messaging_channels.py tests/test_webhooks.py
pytest tests/test_profile_rotation.py tests/test_credential_pool.py
```

**Test Coverage**:

| Suite | Tests | Coverage |
|---|---|---|
| `test_agent_loop.py` | PAORR loop, state transitions | Agent core |
| `test_agent_router.py` | Per-channel routing | Multi-agent |
| `test_canvas.py` | A2UI protocol, server, tool | Canvas |
| `test_credential_pool.py` | Key rotation, cooldown | LLM layer |
| `test_credential_rotation.py` | Profile-based rotation | LLM layer |
| `test_cron.py` | Scheduling, YAML persistence | Cron |
| `test_cron_enhanced.py` | Enhanced cron with channels | Cron |
| `test_messaging.py` | Telegram/Discord/Slack | Messaging |
| `test_messaging_channels.py` | All 12+ channels | Messaging |
| `test_nodes.py` | Device nodes protocol | Canvas Nodes |
| `test_paorr.py` | Plan/Act/Observe/Reflect/Retry | Agent core |
| `test_profile_rotation.py` | Model failover profiles | LLM layer |
| `test_sandbox.py` | Docker/SSH/OpenShell | Sandbox |
| `test_session_db.py` | SQLite persistence | Memory |
| `test_session_tools.py` | Session CLI commands | Sessions |
| `test_skills.py` | SkillEngine scoring | Skills |
| `test_ssh.py` | SSH server + restricted shell | SSH |
| `test_tool_dispatch.py` | Tool dispatch + retry | Tools |
| `test_voice.py` | TTS + wake detection | Voice |
| `test_webhooks.py` | HMAC + management | Webhooks |

> **All 210 tests passing, 2 skipped** (skip conditions: no Docker for SSH tests, no audio hardware for voice tests)

---

<br>

---

## License

ManusClaw is licensed under the **Modified MIT License**. This means:

| Use Case | Credit Required? | Details |
|---|---|---|
| **Personal / Educational** | No | Free to use, modify, distribute — no attribution needed |
| **Internal / Non-Commercial** | No | Free for organizations for non-revenue purposes |
| **Open-Source Projects** | No | Free under any OSI-approved license |
| **Academic / Research** | No | Free for research and publication |
| **Commercial / SaaS** | **YES** | Must credit: *"This product is built upon ManusClaw, created by The-JDdev (SHS Lab)."* |
| **Name / Trademark Use** | **Prohibited** | Cannot use "ManusClaw", "The-JDdev", or "SHS Lab" to market derived products without written permission |

See the [LICENSE](https://github.com/ManusAgents/manusclaw/blob/main/LICENSE) file for full legal text.

---

<br>

---

## Connect with the Developer

<div align="center">

<table>
<tr>
<td align="center">
<a href="https://facebook.com/itsshsshobuj">
<img src="https://img.shields.io/badge/Facebook-itsshsshobuj-1877F2?style=for-the-badge&logo=facebook&logoColor=white" alt="Facebook">
</a>
</td>
<td align="center">
<a href="https://t.me/singularityos">
<img src="https://img.shields.io/badge/Telegram-@singularityos-26A5E4?style=for-the-badge&logo=telegram&logoColor=white" alt="Telegram">
</a>
</td>
<td align="center">
<a href="mailto:thejddev.official@gmail.com">
<img src="https://img.shields.io/badge/Email-thejddev.official@gmail.com-EA4335?style=for-the-badge&logo=gmail&logoColor=white" alt="Email">
</a>
</td>
</tr>
</table>

<br>

<img src="https://img.shields.io/badge/Built_by-The--JDdev_(SHS_Lab)-6C63FF?style=for-the-badge" alt="Developer">
<img src="https://img.shields.io/badge/Also_known_as-SHS_Shobuj-00C853?style=for-the-badge" alt="Alias">

</div>

---

## Support the Project

> If ManusClaw has empowered your workflow, consider supporting its continued development.

| Method | Address |
|---|---|
| **Webmoney (USD)** | Z430378899900 |
| **Webmoney (USDT)** | T202226490170 |
| **USDT (TRC20)** | TH75J4zaMPwhyR3QxEFdwTCgU2Pp3yPUEr |
| **Bkash (BD)** | 01310211442 |

---

<div align="center">

```
  ██╗      ██████╗  ██████╗ ██████╗ ███████╗
  ██║     ██╔════╝ ██╔═══██╗██╔══██╗██╔════╝
  ██║     ██║  ███╗██║   ██║██████╔╝███████╗
  ██║     ██║   ██║██║   ██║██╔═══╝ ╚════██║
  ███████╗╚██████╔╝╚██████╔╝██║     ███████║
  ╚══════╝ ╚═════╝  ╚═════╝ ╚═╝     ╚══════╝
              Claws of Intelligence
```

**Made by [The-JDdev](https://github.com/The-JDdev) — SHS Lab**

**Setup Guide v5.0.0**

</div>
