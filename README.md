# ManusClaw AI Framework — Setup Documentation

> **Version:** v4.0.0  
> **Repository:** [The-JDdev/manusclaw](https://github.com/The-JDdev/manusclaw)  
> **License:** See repository for details

---

## What is ManusClaw?

ManusClaw is a powerful, extensible AI agent framework built in Python that provides a unified interface for interacting with multiple large language model (LLM) providers. It enables you to build, deploy, and manage AI-powered agents that can execute tasks, write code, search the web, manage files, and much more — all from an interactive command-line shell or in headless server mode.

ManusClaw supports a wide range of LLM providers out of the box, including OpenAI, Anthropic, Google, Mistral, AWS Bedrock, Ollama, GGUF, HuggingFace, and Universal/OpenRouter. Whether you want to run models locally through Ollama or connect to cloud-based APIs, ManusClaw has you covered.

### Key Features

- **Multi-provider support** — Seamlessly switch between OpenAI, Anthropic, Google, Mistral, Bedrock, Ollama, GGUF, HuggingFace, and OpenRouter
- **Interactive shell** — Full-featured REPL with auto-completion, syntax highlighting, and slash commands
- **Persistent task system** — Run tasks in the background, manage queues, and resume sessions
- **Memory system** — Built-in memory management with MEMORY.md and USER.md for persistent context
- **Skills system** — Extend ManusClaw with custom skills and tools
- **Docker support** — Run ManusClaw in containers for reproducible, isolated environments
- **Search integration** — Built-in web search via DuckDuckGo and other providers
- **Permission modes** — PLAN mode for safe planning, BUILD mode for full execution
- **Cron scheduling** — Schedule recurring tasks with the built-in cron system
- **Multi-agent orchestration** — Run multiple agent instances with manusclaw-multi

---

## Documentation Index

This repository contains comprehensive setup and usage documentation for ManusClaw. Each document is written to be beginner-friendly while still providing the depth that advanced users need. Every guide includes copy-paste-ready commands, detailed explanations, and troubleshooting tips.

### Getting Started

| Document | Description |
|----------|-------------|
| **[Installation Guide](docs/installation.md)** | Step-by-step installation instructions for every platform: Linux, macOS, Windows, Docker, Termux, Colab, and VPS/cloud providers. Includes prerequisite setup, Python configuration, and verification steps. |
| **[Configuration Guide](docs/configuration.md)** | Complete reference for configuring ManusClaw. Covers config.toml, .env files, all LLM provider setups, API key management, credential pools, search engines, token budgets, permission modes, and environment variables. |
| **[Usage Guide](docs/usage.md)** | How to use ManusClaw day-to-day. Interactive shell, single-shot mode, all slash commands, background tasks, task queues, persistent tasks, session management, the memory system, skills, and the complete tool reference. |

### Deployment & Operations

| Document | Description |
|----------|-------------|
| **[Deployment Guide](docs/deployment.md)** | Production deployment walkthrough: Docker, docker-compose, VPS setup, systemd services, Nginx reverse proxy, SSL/TLS, auto-start, process management with supervisord, security hardening, resource planning, and scaling. |

### Troubleshooting & Maintenance

| Document | Description |
|----------|-------------|
| **[Troubleshooting Guide](docs/troubleshooting.md)** | Solutions to common problems: installation errors, Python version issues, dependency conflicts, API key problems, Ollama connectivity, Playwright issues, permissions, memory errors, network/firewall, platform-specific quirks, rate limiting, token budgets, and clean reinstall. |
| **[Uninstall Guide](docs/uninstall.md)** | Complete removal instructions: pip uninstall, config cleanup, workspace removal, Docker image cleanup, and full data purge. |

### Platform-Specific Guides

| Document | Description |
|----------|-------------|
| **[Termux (Android) Guide](docs/platforms/termux.md)** | Detailed setup for running ManusClaw on Android devices via Termux, including proot/chroot considerations and performance tips. |
| **[Google Colab Guide](docs/platforms/colab.md)** | Step-by-step Colab notebook setup for running ManusClaw in the cloud with free GPU access, including ngrok tunneling for remote access. |
| **[WSL2 Guide](docs/platforms/wsl.md)** | Windows Subsystem for Linux 2 deep-dive, covering installation, GUI support, file system performance, and integration with Windows tools. |

---

## Quick Start

If you just want to get running as fast as possible, here's the absolute minimum:

```bash
# 1. Ensure Python 3.11+ is installed
python3 --version

# 2. Install ManusClaw
pip install manusclaw

# 3. Set your API key (example: OpenAI)
export OPENAI_API_KEY="sk-your-key-here"

# 4. Launch ManusClaw
manusclaw
```

For detailed instructions tailored to your specific platform and use case, follow the **[Installation Guide](docs/installation.md)**.

---

## System Requirements

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| **Python** | 3.11 | 3.12+ |
| **RAM** | 512 MB | 2 GB+ |
| **Disk Space** | 500 MB | 2 GB+ (with Playwright browsers) |
| **OS** | Linux, macOS, Windows | Linux (Ubuntu 22.04+) |
| **Network** | Required for API providers | Stable broadband |

> **Note:** If you plan to run local models via Ollama, you will need significantly more RAM and ideally a GPU. See the [Configuration Guide](docs/configuration.md) for Ollama setup details.

---

## Project Structure

When you install and run ManusClaw, the following directory structure is created:

```
~/.manusclaw/              # Configuration directory
├── config.toml            # Main configuration file
├── .env                   # Environment variables (API keys)
├── MEMORY.md              # Agent persistent memory
├── USER.md                # User profile and preferences
└── skills/                # Custom skills directory

workspace/                 # Default working directory
├── ...                    # Your project files
```

---

## Contributing

Found an error or want to improve these docs? Contributions are welcome! Please open an issue or pull request at the [main repository](https://github.com/The-JDdev/manusclaw).

---

## Support

- **GitHub Issues:** [The-JDdev/manusclaw/issues](https://github.com/The-JDdev/manusclaw/issues)
- **Documentation:** You're looking at it! Start with the [Installation Guide](docs/installation.md).
- **Troubleshooting:** Check the [Troubleshooting Guide](docs/troubleshooting.md) before opening an issue.
