# Installation Guide — ManusClaw v5.1.0

This guide covers every platform and method for installing ManusClaw v5.1.0. Each section is self-contained so you can jump directly to your platform. If you encounter any issues, consult the [Troubleshooting Guide](troubleshooting.md) before opening a GitHub issue.

**What's new in v5.1.0:** Security configuration (RBAC, input validation, audit logging), hooks system, context management, conversation management, observability (OpenTelemetry), secrets management (Vault), file store (S3/GCS/Azure), git providers (GitHub/GitLab/Bitbucket), integrations plugin system, parallel executor, migration system, 15+ messaging channels (Teams/Google Chat/LINE now full), voice v2 with streaming STT, Canvas v2 with collaborative editing, enhanced cron with dependencies and retry policies, Kubernetes Helm chart, and cloud deployment guides (AWS ECS, GCP Cloud Run, Azure Container Instances). See the [Changelog](https://github.com/ManusAgents/manusclaw/releases/tag/v5.1.0) for the full list of changes.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Quick Start (one-liner)](#quick-start-one-liner)
- [Optional Dependency Groups](#optional-dependency-groups)
  - [Dependency Group Reference Table](#dependency-group-reference-table)
  - [Installing All Optional Dependencies](#installing-all-optional-dependencies)
  - [Installing Individual Groups](#installing-individual-groups)
  - [Dependency Tree (v5.1.0)](#dependency-tree-v510)
- [Linux Installation](#linux-installation)
  - [Ubuntu / Debian](#ubuntu--debian)
  - [Fedora / RHEL / CentOS](#fedora--rhel--centos)
  - [Arch Linux / Manjaro](#arch-linux--manjaro)
- [macOS Installation](#macos-installation)
  - [Intel Macs](#intel-macs)
  - [Apple Silicon (M1/M2/M3/M4)](#apple-silicon-m1m2m3m4)
- [Windows Installation](#windows-installation)
  - [Native Windows](#native-windows)
  - [WSL2 (Recommended)](#wsl2-recommended)
- [Docker Installation](#docker-installation)
  - [Docker Compose Profiles](#docker-compose-profiles)
  - [Docker Compose Profiles Explained](#docker-compose-profiles-explained)
- [Upgrading from v5.0.0](#upgrading-from-v500)
- [Verifying the Installation](#verifying-the-installation)
- [Next Steps](#next-steps)

---

## Prerequisites

| Requirement | Minimum | Recommended |
|------------|---------|-------------|
| **Python** | 3.11 | 3.12+ |
| **pip** | 24.0 | Latest |
| **Git** | 2.30 | Latest |
| **RAM** | 2 GB | 4 GB+ |
| **Disk** | 500 MB | 2 GB+ |
| **OS** | Linux, macOS, Windows (WSL2) | Ubuntu 22.04+ / macOS 14+ |

### Additional Prerequisites for v5.1.0 Optional Features

| Feature | Additional Requirement |
|---------|----------------------|
| Voice (wake/talk) | PortAudio (`portaudio19-dev`), PyAudio |
| Canvas v2 | WebSocket server running |
| Docker sandbox | Docker Engine 20.10+ |
| SSH sandbox | SSH access to target host |
| OpenTelemetry | `opentelemetry-api`, `opentelemetry-sdk` |
| Vault secrets | `hvac` (HashiCorp Vault client) |
| S3 file store | `boto3` |
| GCS file store | `google-cloud-storage` |
| Azure Blob file store | `azure-storage-blob` |
| Git providers | `PyGithub`, `python-gitlab`, `atlassian-python-api` |
| Parallel executor | `concurrent-futures` (stdlib), optional `ray` for distributed |

---

## Quick Start (one-liner)

```bash
pip install "manusclaw[all]" && manusclaw --version
```

This installs ManusClaw with all optional dependencies. If you prefer a minimal install:

```bash
pip install manusclaw
```

---

## Optional Dependency Groups

ManusClaw v5.1.0 organizes optional dependencies into groups so you can install only what you need.

### Dependency Group Reference Table

| Group | Install Command | What It Includes |
|-------|----------------|-----------------|
| `voice` | `pip install "manusclaw[voice]"` | PyAudio, Porcupine, speech_recognition, pyttsx3, ElevenLabs SDK |
| `browser` | `pip install "manusclaw[browser]"` | Playwright, Chromium browser |
| `desktop` | `pip install "manusclaw[desktop]"` | Flet (desktop GUI), rumps (macOS menu bar), pystray (Windows tray) |
| `channels` | `pip install "manusclaw[channels]"` | python-telegram-bot, discord.py, slack-sdk, websockets, matrix-client |
| `enterprise` | `pip install "manusclaw[enterprise]"` | hvac (Vault), opentelemetry-api/sdk, PyGithub, python-gitlab, atlassian-python-api |
| `cloud` | `pip install "manusclaw[cloud]"` | boto3 (S3), google-cloud-storage, azure-storage-blob |
| `observability` | `pip install "manusclaw[observability]"` | opentelemetry-api, opentelemetry-sdk, opentelemetry-exporter-otlp, prometheus-client |
| `vault` | `pip install "manusclaw[vault]"` | hvac (HashiCorp Vault client) |
| `git` | `pip install "manusclaw[git]"` | PyGithub, python-gitlab, atlassian-python-api |
| `parallel` | `pip install "manusclaw[parallel]"` | ray (distributed execution), psutil |
| `all` | `pip install "manusclaw[all]"` | Everything above |

### Installing All Optional Dependencies

```bash
pip install "manusclaw[all]"
```

### Installing Individual Groups

```bash
# Voice features only
pip install "manusclaw[voice]"

# Enterprise features (vault, observability, git providers)
pip install "manusclaw[enterprise]"

# Cloud storage backends
pip install "manusclaw[cloud]"

# Combine multiple groups
pip install "manusclaw[voice,enterprise,cloud]"
```

### Dependency Tree (v5.1.0)

```
manusclaw[all]
├── core (always installed)
│   ├── pydantic >= 2.0
│   ├── pyyaml
│   ├── toml
│   ├── httpx
│   ├── rich
│   ├── prompt_toolkit
│   ├── python-dotenv
│   └── aiofiles
├── [voice]
│   ├── pyaudio
│   ├── pvporcupine
│   ├── SpeechRecognition
│   ├── pyttsx3
│   └── elevenlabs
├── [browser]
│   └── playwright
├── [desktop]
│   ├── flet
│   ├── rumps (macOS only)
│   └── pystray (Windows only)
├── [channels]
│   ├── python-telegram-bot
│   ├── discord.py
│   ├── slack-sdk
│   ├── websockets
│   └── matrix-client
├── [enterprise]
│   ├── hvac
│   ├── opentelemetry-api
│   ├── opentelemetry-sdk
│   ├── PyGithub
│   ├── python-gitlab
│   └── atlassian-python-api
├── [cloud]
│   ├── boto3
│   ├── google-cloud-storage
│   └── azure-storage-blob
├── [observability]
│   ├── opentelemetry-api
│   ├── opentelemetry-sdk
│   ├── opentelemetry-exporter-otlp
│   └── prometheus-client
├── [vault]
│   └── hvac
├── [git]
│   ├── PyGithub
│   ├── python-gitlab
│   └── atlassian-python-api
├── [parallel]
│   ├── ray
│   └── psutil
└── shared-optional
    ├── asyncssh (SSH sandbox/gateway)
    ├── docker (Docker sandbox)
    ├── google-api-python-client (Gmail)
    └── cryptography (secrets encryption)
```

---

## Linux Installation

### Ubuntu / Debian

```bash
# 1. Update system packages
sudo apt update && sudo apt upgrade -y

# 2. Install system dependencies
sudo apt install -y python3 python3-pip python3-venv git \
  build-essential portaudio19-dev libffi-dev libssl-dev \
  libcairo2-dev libjpeg-dev libpng-dev

# 3. (Optional) Install additional dependencies for v5.1 enterprise features
sudo apt install -y libgpgme-dev libdevmapper-dev

# 4. Create a virtual environment (recommended)
python3 -m venv ~/manusclaw-env
source ~/manusclaw-env/bin/activate

# 5. Install ManusClaw
pip install "manusclaw[all]"

# 6. (Optional) Install Playwright browsers
playwright install chromium

# 7. Verify
manusclaw --version
```

### Fedora / RHEL / CentOS

```bash
# 1. Update system packages
sudo dnf update -y

# 2. Install system dependencies
sudo dnf install -y python3 python3-pip python3-devel git \
  gcc gcc-c++ make portaudio-devel libffi-devel openssl-devel \
  cairo-devel libjpeg-devel zlib-devel

# 3. Create a virtual environment
python3 -m venv ~/manusclaw-env
source ~/manusclaw-env/bin/activate

# 4. Install ManusClaw
pip install "manusclaw[all]"

# 5. Verify
manusclaw --version
```

### Arch Linux / Manjaro

```bash
# 1. Update system packages
sudo pacman -Syu --noconfirm

# 2. Install system dependencies
sudo pacman -S --noconfirm python python-pip git base-devel \
  portaudio libffi openssl cairo libjpeg-turbo

# 3. Create a virtual environment
python -m venv ~/manusclaw-env
source ~/manusclaw-env/bin/activate

# 4. Install ManusClaw
pip install "manusclaw[all]"

# 5. Verify
manusclaw --version
```

---

## macOS Installation

### Intel Macs

```bash
# 1. Install Homebrew (if not installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 2. Install system dependencies
brew install python@3.12 git portaudio

# 3. Create a virtual environment
python3 -m venv ~/manusclaw-env
source ~/manusclaw-env/bin/activate

# 4. Install ManusClaw
pip install "manusclaw[all]"

# 5. Verify
manusclaw --version
```

### Apple Silicon (M1/M2/M3/M4)

```bash
# 1. Install Homebrew (if not installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 2. Add Homebrew to PATH (Apple Silicon specific)
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"

# 3. Install system dependencies
brew install python@3.12 git portaudio

# 4. Create a virtual environment
python3 -m venv ~/manusclaw-env
source ~/manusclaw-env/bin/activate

# 5. Install ManusClaw
pip install "manusclaw[all]"

# 6. Verify
manusclaw --version
```

> **Note for Apple Silicon:** Some Python packages with C extensions may need to be compiled for `arm64`. If you encounter architecture errors, ensure you're using the ARM64 Homebrew (`/opt/homebrew`) and not the Rosetta Intel Homebrew (`/usr/local`).

---

## Windows Installation

### Native Windows

```powershell
# 1. Install Python 3.12+ from https://www.python.org/downloads/
#    Make sure to check "Add Python to PATH" during installation

# 2. Open PowerShell or Command Prompt

# 3. Create a virtual environment
python -m venv manusclaw-env
.\manusclaw-env\Scripts\Activate.ps1

# 4. Install ManusClaw
pip install "manusclaw[all]"

# 5. Verify
manusclaw --version
```

> **Known limitations on native Windows:** Some features (voice wake word with Porcupine, Docker sandbox, SSH gateway) have limited or no support on native Windows. For the best experience, use WSL2.

### WSL2 (Recommended)

See the dedicated [WSL2 Guide](platforms/wsl.md) for detailed instructions. Quick version:

```bash
# Inside WSL2 (Ubuntu)
sudo apt update && sudo apt install -y python3 python3-pip python3-venv git portaudio19-dev
python3 -m venv ~/manusclaw-env
source ~/manusclaw-env/bin/activate
pip install "manusclaw[all]"
manusclaw --version
```

---

## Docker Installation

### Basic Docker Run

```bash
docker run -it --rm \
  -e OPENAI_API_KEY=sk-proj-your-key \
  -v ~/.manusclaw:/root/.manusclaw \
  manusclaw/manusclaw:5.1.0
```

### Docker with All v5.1 Features

```bash
docker run -it --rm \
  -e OPENAI_API_KEY=sk-proj-your-key \
  -e ANTHROPIC_API_KEY=sk-ant-your-key \
  -e VAULT_ADDR=http://vault:8200 \
  -e VAULT_TOKEN=your-vault-token \
  -e OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317 \
  -v ~/.manusclaw:/root/.manusclaw \
  -p 8765:8765 \
  -p 2222:2222 \
  manusclaw/manusclaw:5.1.0
```

### Docker Compose Profiles

The v5.1.0 Docker Compose file uses profiles to let you start only the services you need:

```yaml
# docker-compose.yml
version: "3.8"

services:
  manusclaw:
    image: manusclaw/manusclaw:5.1.0
    profiles: ["core"]
    ports:
      - "8765:8765"
    volumes:
      - ./config:/root/.manusclaw
    env_file: .env

  manusclaw-ssh:
    image: manusclaw/manusclaw:5.1.0
    profiles: ["ssh"]
    command: manusclaw-ssh start
    ports:
      - "2222:2222"
    volumes:
      - ./config:/root/.manusclaw
    env_file: .env

  manusclaw-telegram:
    image: manusclaw/manusclaw:5.1.0
    profiles: ["channels"]
    command: manusclaw-channels start telegram
    volumes:
      - ./config:/root/.manusclaw
    env_file: .env

  manusclaw-discord:
    image: manusclaw/manusclaw:5.1.0
    profiles: ["channels"]
    command: manusclaw-channels start discord
    volumes:
      - ./config:/root/.manusclaw
    env_file: .env

  manusclaw-cron:
    image: manusclaw/manusclaw:5.1.0
    profiles: ["cron"]
    command: manusclaw-cron start
    volumes:
      - ./config:/root/.manusclaw
    env_file: .env

  vault:
    image: hashicorp/vault:1.15
    profiles: ["enterprise"]
    ports:
      - "8200:8200"
    environment:
      VAULT_DEV_ROOT_TOKEN_ID: "dev-only-token"
    cap_add:
      - IPC_LOCK

  otel-collector:
    image: otel/opentelemetry-collector-contrib:0.96.0
    profiles: ["observability"]
    ports:
      - "4317:4317"
      - "4318:4318"
    volumes:
      - ./otel-config.yaml:/etc/otelcol-contrib/config.yaml

  prometheus:
    image: prom/prometheus:v2.50.0
    profiles: ["observability"]
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
```

### Docker Compose Profiles Explained

| Profile | Services | Use Case |
|---------|----------|----------|
| `core` | manusclaw | Basic agent server |
| `ssh` | manusclaw-ssh | SSH remote gateway |
| `channels` | manusclaw-telegram, manusclaw-discord | Messaging channel adapters |
| `cron` | manusclaw-cron | Scheduled task runner |
| `enterprise` | vault | HashiCorp Vault for secrets |
| `observability` | otel-collector, prometheus | OpenTelemetry + metrics |

```bash
# Start core only
docker compose --profile core up -d

# Start core + SSH + channels
docker compose --profile core --profile ssh --profile channels up -d

# Start everything
docker compose --profile core --profile ssh --profile channels \
  --profile cron --profile enterprise --profile observability up -d
```

---

## Upgrading from v5.0.0

### In-Place Upgrade

```bash
# Stop any running ManusClaw processes
pkill -f manusclaw

# Upgrade the package
pip install --upgrade "manusclaw[all]"

# Run the migration system to upgrade config
manusclaw-migrate --from 5.0.0 --to 5.1.0

# Verify
manusclaw --version
```

### What the Migration Does

The `manusclaw-migrate` command handles:

1. **Config schema migration** — Adds new v5.1.0 sections to `config.yaml` and `config.toml` with safe defaults
2. **Session format migration** — Updates session data to the new v5.1.0 format
3. **Cron job migration** — Migrates `cron.yaml` to the new format with retry policies
4. **State directory migration** — Creates new directories for v5.1.0 features (hooks, secrets, migrations)

### Manual Migration Steps

If you prefer to migrate manually:

```bash
# 1. Back up your existing configuration
cp -r ~/.manusclaw ~/.manusclaw-v5.0-backup

# 2. Create new v5.1.0 config directories
mkdir -p ~/.manusclaw/hooks
mkdir -p ~/.manusclaw/secrets
mkdir -p ~/.manusclaw/migrations
mkdir -p ~/.manusclaw/context
mkdir -p ~/.manusclaw/conversations

# 3. Update config.yaml with new sections (see Configuration Guide)
# Add: security, hooks, context, conversation, observability, secrets,
#      file_store, git_providers, integrations, parallel_executor, migrations

# 4. Verify the config is valid
manusclaw --validate-config
```

---

## Verifying the Installation

After installing, verify everything is working:

```bash
# Check version
manusclaw --version
# Expected: manusclaw 5.1.0

# Check config is valid
manusclaw --validate-config

# Run a quick test
manusclaw "What is 2 + 2?"
# Expected: 4

# Check available optional features
manusclaw --features
# Lists all features and their availability status

# Start the server and check health
manusclaw-server &
curl http://localhost:8765/health
```

---

## Next Steps

- **Configure your LLM provider**: See [Configuration Guide](configuration.md)
- **Explore features**: See [Features Guide](features.md)
- **Deploy to production**: See [Deployment Guide](deployment.md)
- **Set up messaging channels**: See [Features Guide → Channels](features.md)
- **Enable enterprise features**: See [Configuration Guide → Security, Observability, Secrets](configuration.md)
