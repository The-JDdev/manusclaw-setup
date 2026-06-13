<div align="center">

<img src="https://img.shields.io/badge/ManusClaw-v5.1.0-ff69b4?style=for-the-badge&logo=github&logoColor=white" alt="Version">
<img src="https://img.shields.io/badge/Setup_Guide-Production_Ready-00C853?style=for-the-badge&logo=book&logoColor=white" alt="Setup Guide">
<img src="https://img.shields.io/badge/Python-3.11+-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python">
<img src="https://img.shields.io/badge/License-Modified_MIT-FFD700?style=for-the-badge&logo=opensourceinitiative&logoColor=black" alt="License">
<img src="https://img.shields.io/badge/Tests-340_Passing-00C853?style=for-the-badge&logo=check-circle&logoColor=white" alt="Tests">
<img src="https://img.shields.io/badge/Features-22%2B_Categories-6C63FF?style=for-the-badge&logo=rocket&logoColor=white" alt="Features">

<br><br>

# MANUSCLAW — THE DEFINITIVE SETUP GUIDE

### _From Zero to Autonomous AI Beast in Minutes_

**Production-grade installation and setup documentation for ManusClaw v5.1.0 — the omnichannel autonomous AI agent framework.**

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

```
manusclaw-setup/
├── README.md                        ← You are here
├── docs/
│   ├── installation.md              Full platform-by-platform install guide
│   ├── configuration.md             Every config option explained (config.yaml, config.toml, .env)
│   ├── features.md                  All 22+ feature categories with usage examples
│   ├── usage.md                     Day-to-day usage: CLI, sessions, skills, tools, slash commands
│   ├── deployment.md                Docker, Kubernetes, VPS, cloud, reverse proxy, SSL
│   ├── troubleshooting.md           Common errors and solutions
│   ├── uninstall.md                 Complete removal guide
│   └── platforms/
│       ├── colab.md                 Google Colab setup
│       ├── termux.md                Android/Termux setup
│       └── wsl.md                   Windows Subsystem for Linux 2
```

---

## What's New in v5.1.0

ManusClaw v5.1.0 is a major feature release that introduces enterprise-grade infrastructure, advanced security controls, and powerful new orchestration capabilities. Here's what's new:

### New Configuration Systems
- **Security Configuration** — RBAC policies, input validation, rate limiting, sandbox restrictions, and audit logging
- **Hooks System** — Pre/post execution lifecycle hooks for custom logic injection
- **Context Management** — Context window optimization, compression, and summarization strategies
- **Conversation Management** — Threaded conversations, persistence backends, and conversation export
- **Observability** — OpenTelemetry integration, distributed tracing, metrics, and structured logging
- **Secrets Management** — HashiCorp Vault integration, secret rotation, and encrypted storage
- **File Store** — Pluggable storage backends (local, S3, GCS, Azure Blob) for agent artifacts
- **Git Providers** — GitHub, GitLab, and Bitbucket integration for repository-aware agent operations
- **Integrations** — Plugin system for third-party services (Jira, Notion, PagerDuty, custom)
- **Parallel Executor** — Concurrent agent execution with worker pools and dependency graphs
- **Migrations** — Database schema migrations and config version upgrade system

### Enhanced Existing Features
- **15+ Messaging Channels** — Added Google Chat (full), Microsoft Teams (full), LINE
- **Voice v2** — Improved wake word detection, streaming STT, voice activity detection
- **Canvas v2** — Real-time collaborative canvas with multi-user editing
- **Enhanced Cron** — Cron job dependencies, retry policies, and distributed scheduling
- **Docker Compose v2** — Production-ready compose with health checks and auto-restart
- **Kubernetes** — Full Helm chart with auto-scaling, rolling updates, and pod disruption budgets
- **Cloud Deployment** — AWS ECS, Google Cloud Run, Azure Container Instances guides

---

## Quick Install

### One-Liner (Linux / macOS / WSL2)

```bash
pip install "manusclaw[all]"
```

### With Specific Dependency Groups

```bash
# Core + voice features
pip install "manusclaw[voice]"

# Core + enterprise features (vault, observability, parallel executor)
pip install "manusclaw[enterprise]"

# Core + all cloud storage backends
pip install "manusclaw[cloud]"

# Everything
pip install "manusclaw[all]"
```

### Docker

```bash
docker run -it --rm \
  -e OPENAI_API_KEY=sk-proj-your-key \
  -v ~/.manusclaw:/root/.manusclaw \
  manusclaw/manusclaw:5.1.0
```

### Verify Installation

```bash
manusclaw --version
# manusclaw 5.1.0
```

---

## Documentation Index

| Document | Description |
|----------|-------------|
| [Installation Guide](docs/installation.md) | Platform-by-platform install instructions (Linux, macOS, Windows, Docker, Colab, Termux) |
| [Configuration Guide](docs/configuration.md) | Every config option: `config.yaml`, `config.toml`, `.env`, profiles, priority chain |
| [Features Guide](docs/features.md) | Complete feature reference for all 22+ categories |
| [Usage Guide](docs/usage.md) | CLI commands, sessions, skills, tools, slash commands, profiles |
| [Deployment Guide](docs/deployment.md) | Docker, Kubernetes, VPS, cloud (AWS/GCP/Azure), reverse proxy, SSL |
| [Troubleshooting Guide](docs/troubleshooting.md) | Common errors and solutions |
| [Uninstall Guide](docs/uninstall.md) | Complete removal instructions |
| [Colab Guide](docs/platforms/colab.md) | Google Colab setup with GPU/Ollama |
| [Termux Guide](docs/platforms/termux.md) | Android/Termux setup |
| [WSL2 Guide](docs/platforms/wsl.md) | Windows Subsystem for Linux 2 setup |

---

## Feature Overview

### Core Agent
- Autonomous task execution with planning, tool use, and verification
- 15+ messaging channels (Telegram, Discord, Slack, WhatsApp, Signal, Matrix, IRC, Twitch, WebChat, Teams, Google Chat, LINE, Email)
- 12+ LLM providers with seamless switching and failover
- Model failover profiles with adaptive cooldown
- Credential pool for API key rotation
- Multi-agent routing and orchestration

### Voice & Canvas
- Voice wake word detection (Porcupine, Google STT, streaming STT)
- Voice talk mode with TTS (ElevenLabs, OpenAI, system, streaming)
- Live Canvas (A2UI) with real-time collaborative editing
- Mobile/desktop node clients

### Enterprise & Infrastructure
- Security policies with RBAC, input validation, and audit logging
- Secrets management with HashiCorp Vault and encrypted storage
- Observability with OpenTelemetry, distributed tracing, and metrics
- Pluggable file storage (local, S3, GCS, Azure Blob)
- Git provider integration (GitHub, GitLab, Bitbucket)
- Parallel executor for concurrent agent execution
- Hooks system for custom lifecycle logic

### Operations
- Docker, Docker Compose, and Kubernetes deployment
- Cloud deployment (AWS ECS, Google Cloud Run, Azure Container Instances)
- SSH remote gateway with command whitelisting
- Webhook management with HMAC verification
- Enhanced cron with dependencies and retry policies
- Gmail Pub/Sub automation
- Session management and export
- Skills system
- Config profiles and migration system

---

## System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **Python** | 3.11 | 3.12+ |
| **RAM** | 2 GB | 4 GB+ |
| **Disk** | 500 MB | 2 GB+ |
| **OS** | Linux, macOS, Windows (WSL2) | Ubuntu 22.04+ / macOS 14+ |

---

## Project Links

- **Core Engine**: [github.com/ManusAgents/manusclaw](https://github.com/ManusAgents/manusclaw)
- **Setup & Docs**: [github.com/The-JDdev/manusclaw-setup](https://github.com/The-JDdev/manusclaw-setup)
- **Changelog**: [github.com/ManusAgents/manusclaw/releases](https://github.com/ManusAgents/manusclaw/releases)
- **Issues**: [github.com/ManusAgents/manusclaw/issues](https://github.com/ManusAgents/manusclaw/issues)

---

## License

ManusClaw is released under a Modified MIT License. See the [core repository](https://github.com/ManusAgents/manusclaw) for details.

---

<p align="center">
  Built by <a href="https://github.com/The-JDdev">The-JDdev</a> (SHS Lab)
</p>
