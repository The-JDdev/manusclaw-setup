# Installation Guide — ManusClaw v5.0.0

This guide covers every platform and method for installing ManusClaw v5.0.0. Each section is self-contained so you can jump directly to your platform. If you encounter any issues, consult the [Troubleshooting Guide](troubleshooting.md) before opening a GitHub issue.

**What's new in v5.0.0:** Voice I/O (Porcupine + ElevenLabs TTS), SSH remote execution, Gmail integration, Matrix messaging, system tray companions, multi-agent Docker Compose profiles, optional dependency groups, and an expanded dependency tree. See the [Changelog](https://github.com/ManusAgents/manusclaw/releases/tag/v5.0.0) for the full list of changes.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Quick Start (one-liner)](#quick-start-one-liner)
- [Optional Dependency Groups](#optional-dependency-groups)
  - [Dependency Group Reference Table](#dependency-group-reference-table)
  - [Installing All Optional Dependencies](#installing-all-optional-dependencies)
  - [Installing Individual Groups](#installing-individual-groups)
  - [Dependency Tree (v5.0.0)](#dependency-tree-v500)
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
  - [Volume Mounts](#docker-volume-mounts)
- [Termux (Android) Installation](#termux-android-installation)
- [Google Colab Installation](#google-colab-installation)
- [VPS / Cloud Server Installation](#vps--cloud-server-installation)
  - [General VPS Setup](#general-vps-setup-applies-to-all-providers)
  - [DigitalOcean](#digitalocean)
  - [AWS EC2](#aws-ec2)
  - [Google Cloud Platform](#google-cloud-platform)
  - [Microsoft Azure](#microsoft-azure)
- [Verification](#verification)
- [Post-Installation Setup](#post-installation-setup)
- [Using the Install Scripts](#using-the-install-scripts)
  - [Linux / macOS — install.sh](#linux--macos---installsh)
  - [Windows — install.ps1](#windows---installps1)
  - [Termux — setup-termux.sh](#termux---setup-termuxsh)
- [Using pyenv (Alternative Python Installation)](#using-pyenv-alternative-python-installation)
- [Virtual Environments](#virtual-environments)
- [Upgrading from v4.0.0](#upgrading-from-v400)

---

## Prerequisites

Before installing ManusClaw v5.0.0, ensure your system meets these baseline requirements:

| Requirement | Details |
|-------------|---------|
| **Python** | **3.11 minimum** (3.12+ recommended; 3.13 supported) |
| **pip** | Latest version (bundled with Python, but should be updated) |
| **Git** | For cloning the repository if installing from source |
| **Internet** | Required for API-based providers and package downloads |
| **Disk Space** | At least 500 MB free (2 GB+ recommended for Playwright browsers; 4 GB+ with voice dependencies) |
| **RAM** | 512 MB minimum (2 GB+ recommended; more for local models or voice features) |
| **Audio** | Microphone + speakers (optional — only needed for `[voice]` group) |

### Why Python 3.11+?

ManusClaw v5.0.0 requires Python 3.11 as the absolute minimum. This is a hard requirement — **Python 3.10 and below will not work**. The reasons are:

- **`tomllib` module** — added in 3.11, used for parsing `pyproject.toml` configuration files throughout ManusClaw
- **`ExceptionGroup` and `except*` syntax** — used in the multi-agent orchestrator for handling concurrent failures
- **`typing.ParamSpec` enhancements** — used in the plugin system for type-safe hook registration
- **Performance** — Python 3.11 includes up to 25% speed improvements over 3.10, and 3.12 adds another 5-10%, which directly benefit agent responsiveness
- **`TaskGroup`** — the structured concurrency primitive used in the SSH and voice subsystems

### Platform Support Matrix

| Platform | Status | Notes |
|----------|--------|-------|
| Ubuntu 22.04+ | ✅ Full | Install Python 3.11 via deadsnakes PPA |
| Ubuntu 24.04+ | ✅ Full | Python 3.12 included by default |
| Debian 12+ | ✅ Full | Python 3.11 included by default |
| Fedora 37+ | ✅ Full | Python 3.12 included by default |
| Arch Linux | ✅ Full | Latest Python always available |
| macOS (Intel) | ✅ Full | Install via Homebrew |
| macOS (Apple Silicon) | ✅ Full | Native ARM64 support |
| Windows 10/11 | ✅ Full | Native or WSL2 |
| WSL2 | ✅ Full (recommended) | Best Windows experience |
| Docker | ✅ Full | Three Compose profiles available |
| Termux (Android) | ✅ Partial | No Playwright; voice may require extra steps |
| Google Colab | ✅ Partial | Single-shot mode; no interactive shell |
| VPS / Cloud | ✅ Full | Any Linux provider |

---

## Quick Start (one-liner)

If you just want to get started and you already have Python 3.11+ and pip installed:

```bash
# Minimum install (core features only)
pip install manusclaw

# Install with ALL optional dependencies (voice, SSH, Gmail, Matrix, companion)
pip install manusclaw[all-plus]
```

Then configure your API key and run:

```bash
export OPENAI_API_KEY="sk-your-key-here"
manusclaw
```

> **New in v5.0.0:** The `[all-plus]` extra installs every optional dependency group in one command. See [Optional Dependency Groups](#optional-dependency-groups) for details.

---

## Optional Dependency Groups

Starting with v5.0.0, ManusClaw uses **PEP 621 dependency groups** to organize optional features. The core package includes everything you need for basic agent operation (LLM interaction, web search, file I/O, Playwright browsing). Additional capabilities are installed via optional extras.

This design keeps the base install lightweight while allowing you to opt in to specific features without pulling in unnecessary dependencies.

### Dependency Group Reference Table

| Extra Name | Install Command | What It Adds |
|-----------|----------------|-------------|
| `voice` | `pip install manusclaw[voice]` | Wake-word detection (Porcupine), speech-to-text (speech_recognition), text-to-speech (pyttsx3, elevenlabs), audio capture (sounddevice) |
| `ssh` | `pip install manusclaw[ssh]` | Remote SSH command execution via asyncssh |
| `gmail` | `pip install manusclaw[gmail]` | Gmail read/send integration via google-api-python-client |
| `matrix` | `pip install manusclaw[matrix]` | Matrix protocol messaging via matrix-nio |
| `companion` | `pip install manusclaw[companion]` | System tray icon / menu bar companion app via pystray (Linux/Windows) or rumps (macOS) |
| `all-plus` | `pip install manusclaw[all-plus]` | **All of the above** in a single install |

### Installing All Optional Dependencies

The `[all-plus]` meta-extra is the simplest way to get everything:

```bash
pip install manusclaw[all-plus]
```

This is equivalent to:

```bash
pip install manusclaw[voice,ssh,gmail,matrix,companion]
```

> **Tip:** Use `[all-plus]` for development and evaluation. For production deployments, install only the groups you actually need to minimize attack surface and container image size.

### Installing Individual Groups

You can install any combination of groups. pip will merge them correctly:

```bash
# Just voice + SSH
pip install manusclaw[voice,ssh]

# Gmail + Matrix for notification workflows
pip install manusclaw[gmail,matrix]

# Add groups to an existing editable install
pip install -e ".[voice,companion]"
```

You can also add groups after the initial install:

```bash
# Core is already installed; add voice support later
pip install manusclaw[voice]
```

pip detects the existing installation and only installs the additional dependencies.

### Dependency Tree (v5.0.0)

Below is the complete dependency tree for ManusClaw v5.0.0. Dependencies marked with `*` are **core** (always installed). Dependencies marked with `[group]` are installed only when that optional group is requested.

#### Core Dependencies (always installed)

```
manusclaw v5.0.0
├── python >= 3.11
├── pip >= 23.0
│
├── * openai >= 1.12.0
├── * anthropic >= 0.25.0
├── * httpx >= 0.27.0
├── * pydantic >= 2.5.0
├── * pydantic-settings >= 2.1.0
├── * tomli (on Python <3.11; stdlib tomllib on 3.11+)
├── * rich >= 13.0.0
├── * click >= 8.1.0
├── * uvicorn >= 0.27.0
├── * fastapi >= 0.110.0
├── * python-dotenv >= 1.0.0
├── * playwright >= 1.42.0
├── * duckduckgo-search >= 6.0.0
├── * pathspec >= 0.12.0
├── * jinja2 >= 3.1.0
├── * aiofiles >= 23.2.0
├── * tenacity >= 8.2.0
└── * tiktoken >= 0.5.0
```

#### Optional Group: `[voice]`

```
manusclaw[voice]
├── * pvporcupine >= 3.0.0         # Wake-word detection (Porcupine)
├── * sounddevice >= 0.4.6         # Cross-platform audio capture/playback
├── * speech-recognition >= 3.10.0 # Speech-to-text (multiple engines)
├── * pyttsx3 >= 2.90              # Offline text-to-speech
└── * elevenlabs >= 1.0.0          # Cloud text-to-speech (ElevenLabs API)
```

#### Optional Group: `[ssh]`

```
manusclaw[ssh]
└── * asyncssh >= 2.14.0           # Async SSH2 protocol client
```

#### Optional Group: `[gmail]`

```
manusclaw[gmail]
└── * google-api-python-client >= 2.120.0  # Gmail API client
```

#### Optional Group: `[matrix]`

```
manusclaw[matrix]
└── * matrix-nio >= 0.24.0         # Matrix client library (async)
```

#### Optional Group: `[companion]`

```
manusclaw[companion]
├── * pystray >= 0.19.0            # System tray (Linux/Windows)
└── * rumps >= 0.4.0               # macOS menu bar helper (Objective-C bridge)
```

#### Transitive Dependencies of Note

Some optional groups pull in additional transitive dependencies that are worth knowing about:

| Transitive Dependency | Pulled In By | Purpose |
|-----------------------|-------------|---------|
| `PyAudio` | `pvporcupine`, `speech_recognition` | Low-level audio I/O on Python |
| `pynput` | `pystray` | Keyboard/mouse monitoring for companion |
| `pyobjc` | `rumps` | Objective-C bridge for macOS menu bar |
| `oauthlib` | `google-api-python-client` | OAuth2 flow for Gmail authentication |
| `google-auth` | `google-api-python-client` | Google service account / user auth |
| `pycryptodome` | `matrix-nio` | End-to-end encryption for Matrix |

---

## Linux Installation

### Ubuntu / Debian

Ubuntu and Debian are the most common platforms for running ManusClaw. These instructions cover Ubuntu 20.04, 22.04, 24.04 and Debian 11 (Bullseye), 12 (Bookworm).

#### Step 1: Update your system

Always start with an up-to-date system to avoid conflicts with stale packages:

```bash
sudo apt update && sudo apt upgrade -y
```

#### Step 2: Install Python 3.11+

**Ubuntu 24.04** ships with Python 3.12 by default — you're already set.

**Ubuntu 22.04** ships with Python 3.10, which is **too old for v5.0.0**. You need 3.11+.

**Debian 12 (Bookworm)** ships with Python 3.11 — you're set.

**Debian 11 (Bullseye)** ships with Python 3.9 — too old. You'll need the steps below.

Check your current version:

```bash
python3 --version
```

If you already have Python 3.11+, skip to Step 3.

**Install Python 3.11 on Ubuntu / older Debian:**

```bash
# Add the deadsnakes PPA (Ubuntu only)
sudo apt install -y software-properties-common
sudo add-apt-repository -y ppa:deadsnakes/ppa
sudo apt update

# Install Python 3.11 and associated packages
sudo apt install -y python3.11 python3.11-venv python3.11-dev

# Set Python 3.11 as the default (optional but recommended)
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 1
```

For **Debian 11**, you may need to build Python from source or use the `pyenv` method described later.

#### Step 3: Install pip and dependencies

```bash
# Install pip
sudo apt install -y python3-pip

# Install system-level dependencies that ManusClaw or its packages may need
sudo apt install -y build-essential libssl-dev libffi-dev \
    python3-dev python3-venv git curl

# Additional dependencies for the [voice] optional group
sudo apt install -y portaudio19-dev libasound2-dev

# Upgrade pip to the latest version
python3 -m pip install --upgrade pip
```

> **Note:** The `portaudio19-dev` and `libasound2-dev` packages are only needed if you plan to install the `[voice]` optional group. They provide the system-level audio libraries required by `pvporcupine` and `speech_recognition`.

#### Step 4: Install ManusClaw

You have two options: install from PyPI (recommended) or install from source.

**Option A: Install from PyPI (recommended)**

```bash
# Core only
pip install manusclaw

# Or with all optional dependencies
pip install manusclaw[all-plus]
```

**Option B: Install from source**

Installing from source gives you access to the latest development changes before they are released on PyPI. This is useful if you want bleeding-edge features or need to contribute code:

```bash
# Clone the repository
git clone https://github.com/ManusAgents/manusclaw.git
cd manusclaw

# Install in editable mode (core only)
pip install -e .

# Or install in editable mode with all optional dependencies
pip install -e ".[all-plus]"

# Or install a specific version/tag
git checkout v5.0.0
pip install -e ".[all-plus]"
```

> **New in v5.0.0:** Source installs support the same optional dependency groups as PyPI installs. Use `pip install -e ".[voice,ssh]"` for a targeted editable install.

#### Step 5: Install Playwright browsers (optional but recommended)

Playwright is used for web browsing and scraping capabilities. Without it, ManusClaw cannot browse the web interactively:

```bash
playwright install chromium
```

If you encounter permission issues, try:

```bash
playwright install --with-deps chromium
```

The `--with-deps` flag automatically installs system-level dependencies that Chromium needs (like libgbm, libnss3, etc.), which is very helpful on minimal server installations.

#### Step 6: Verify installation

```bash
manusclaw --version
```

You should see output like `ManusClaw v5.0.0`. If you see an error, consult the [Troubleshooting Guide](troubleshooting.md).

#### Step 7: Verify optional groups (if installed)

If you installed the `[voice]` group, verify audio capture:

```bash
python3 -c "import sounddevice; print('Audio devices:', sounddevice.query_devices())"
```

If you installed the `[ssh]` group, verify asyncssh:

```bash
python3 -c "import asyncssh; print('asyncssh', asyncssh.__version__)"
```

---

### Fedora / RHEL / CentOS

Fedora typically ships with very recent Python versions, making installation straightforward. RHEL and CentOS may require additional repositories.

#### Step 1: Update your system

```bash
sudo dnf update -y
```

#### Step 2: Install Python 3.11+

**Fedora 39+** ships with Python 3.12+ by default. Check your version:

```bash
python3 --version
```

If you have Python 3.11+, proceed to Step 3.

**RHEL 8/9 and CentOS Stream** may need the AppStream repository:

```bash
# RHEL 9
sudo dnf install -y python3.11 python3.11-pip python3.11-devel

# RHEL 8 (enable the module first)
sudo dnf module enable -y python3.11
sudo dnf install -y python3.11 python3.11-pip python3.11-devel
```

#### Step 3: Install build dependencies

```bash
sudo dnf install -y gcc gcc-c++ make openssl-devel libffi-devel \
    python3-devel git curl

# Optional: for [voice] group
sudo dnf install -y portaudio-devel alsa-lib-devel
```

#### Step 4: Install ManusClaw

```bash
# Upgrade pip
python3 -m pip install --upgrade pip

# Install ManusClaw (core)
pip install manusclaw

# Or with all optional dependencies
pip install manusclaw[all-plus]
```

#### Step 5: Install Playwright browsers

```bash
playwright install --with-deps chromium
```

On RHEL/CentOS, the `--with-deps` flag is especially important because minimal installations often lack the graphics libraries that Chromium requires.

#### Step 6: Verify

```bash
manusclaw --version
```

---

### Arch Linux / Manjaro

Arch and its derivatives always ship with the latest Python, making this the easiest Linux installation.

#### Step 1: Update your system

```bash
sudo pacman -Syu
```

#### Step 2: Install Python and dependencies

```bash
sudo pacman -S --needed python python-pip python-virtualenv git curl base-devel

# Optional: for [voice] group
sudo pacman -S --needed portaudio
```

Arch's Python is typically the latest stable release (3.12+ at the time of writing), so you should be all set. Verify:

```bash
python3 --version
```

#### Step 3: Install ManusClaw

```bash
# Core only
pip install manusclaw

# Or with all optional dependencies
pip install manusclaw[all-plus]
```

If you prefer to use Arch's AUR (Arch User Repository), check if a `manusclaw` package exists:

```bash
# Using yay
yay -S manusclaw

# Using paru
paru -S manusclaw
```

> **Note:** AUR packages are community-maintained and may not always be up to date. The PyPI method (`pip install`) is the most reliable way to get the latest version. AUR packages also may not support the `[all-plus]` extras syntax.

#### Step 4: Install Playwright browsers

```bash
playwright install --with-deps chromium
```

#### Step 5: Verify

```bash
manusclaw --version
```

---

## macOS Installation

macOS installation is straightforward, but you need to choose the right method based on your Mac's architecture. Intel Macs use x86_64 binaries, while Apple Silicon Macs (M1/M2/M3/M4) use ARM64 binaries. This affects which Homebrew packages and Python builds you'll use.

### Intel Macs

#### Step 1: Install Homebrew

Homebrew is the de facto package manager for macOS. If you don't have it yet:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Follow the on-screen instructions. Homebrew will install the Xcode Command Line Tools automatically if they're not already present.

#### Step 2: Install Python 3.11+

```bash
brew install python@3.11
```

After installation, verify the version:

```bash
python3.11 --version
```

If you want `python3` to point to 3.11 by default, you can create an alias in your shell profile:

```bash
echo 'alias python3=python3.11' >> ~/.zshrc
source ~/.zshrc
```

Or use Homebrew's linking:

```bash
brew link python@3.11
```

#### Step 3: Install ManusClaw

```bash
python3.11 -m pip install --upgrade pip

# Core only
pip3.11 install manusclaw

# Or with all optional dependencies
pip3.11 install manusclaw[all-plus]
```

> **macOS Note for `[companion]` group:** On macOS, the companion app uses `rumps` (built on PyObjC) which requires Xcode Command Line Tools. These are typically installed automatically by Homebrew. If you get errors, run `xcode-select --install`.

#### Step 4: Install Playwright browsers

```bash
playwright install chromium
```

#### Step 5: Verify

```bash
manusclaw --version
```

---

### Apple Silicon (M1/M2/M3/M4)

Apple Silicon Macs provide excellent performance for running ManusClaw, especially if you use Ollama to run local models on the Neural Engine / GPU. The v5.0.0 voice features also benefit from Apple Silicon's efficient audio processing.

#### Step 1: Install Homebrew (if not already installed)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

> **Important:** On Apple Silicon Macs, Homebrew installs to `/opt/homebrew` instead of `/usr/local`. After installation, make sure your PATH is configured correctly. The installer will show you the exact commands to run, which typically look like:

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

#### Step 2: Install Python 3.11+

```bash
brew install python@3.11
```

Homebrew on Apple Silicon installs native ARM64 Python, which gives you the best performance. Verify:

```bash
python3.11 --version
# Should show Python 3.11.x

# Verify it's ARM64 (not running under Rosetta)
file $(which python3.11)
# Should mention "arm64" or "Mach-O 64-bit executable arm64"
```

#### Step 3: Install ManusClaw

```bash
python3.11 -m pip install --upgrade pip

# Core only
pip3.11 install manusclaw

# Or with all optional dependencies
pip3.11 install manusclaw[all-plus]
```

#### Step 4: Install Playwright browsers

```bash
playwright install chromium
```

Playwright automatically downloads the ARM64 version of Chromium on Apple Silicon Macs.

#### Step 5: (Optional) Install Ollama for local models

Apple Silicon's unified memory architecture makes it exceptionally good at running local LLMs. To take advantage of this:

```bash
brew install ollama
ollama serve &
ollama pull llama3
```

See the [Configuration Guide](configuration.md) for details on configuring ManusClaw to use Ollama.

#### Step 6: (Optional) Set up voice features on macOS

The `[voice]` group works out of the box on macOS with the built-in microphone. For the Porcupine wake-word engine, you'll need a Picovoice access key (free tier allows 3 wake words):

```bash
export PORCUPINE_ACCESS_KEY="your-access-key-here"
```

Add this to your `~/.zshrc` or `~/.manusclaw/.env` for persistence.

#### Step 7: Verify

```bash
manusclaw --version
```

---

## Windows Installation

Windows users have two options: native installation or WSL2. **WSL2 is strongly recommended** because it provides a Linux environment that is more compatible with ManusClaw's ecosystem and avoids common Windows-specific issues (path separators, shell differences, permission quirks). However, native Windows installation works fine for most use cases.

### Native Windows

#### Step 1: Install Python 3.11+

1. Download Python 3.11+ from [python.org](https://www.python.org/downloads/)
2. Run the installer
3. **Critical:** Check the box that says **"Add python.exe to PATH"** at the bottom of the installer
4. Click "Install Now" (or "Customize installation" if you want to choose the install location)
5. After installation, open a **new** Command Prompt or PowerShell window and verify:

```powershell
python --version
# Should show Python 3.11.x or higher

pip --version
# Should show pip 24.x or higher
```

If `python` is not recognized, you need to add it to your PATH manually:

1. Open "Edit the system environment variables" from the Start menu
2. Click "Environment Variables"
3. Under "System variables", find "Path" and click "Edit"
4. Add these paths (adjust for your Python version and install location):
   - `C:\Users\YourName\AppData\Local\Programs\Python\Python311\`
   - `C:\Users\YourName\AppData\Local\Programs\Python\Python311\Scripts\`
5. Click OK and restart your terminal

#### Step 2: Install Git

Download Git from [git-scm.com](https://git-scm.com/download/win) and run the installer with default settings.

#### Step 3: Install ManusClaw

```powershell
# Upgrade pip
python -m pip install --upgrade pip

# Core only
pip install manusclaw

# Or with all optional dependencies
pip install manusclaw[all-plus]
```

#### Step 4: Install Playwright browsers

```powershell
playwright install chromium
```

> **Note:** On Windows, Playwright may require additional Visual C++ redistributables. If you get an error about missing DLLs, install the [Microsoft Visual C++ Redistributable](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist).

#### Step 5: Verify

```powershell
manusclaw --version
```

#### Step 6: Windows-specific notes for optional groups

**`[voice]` group on Windows:**
The `pyttsx3` TTS engine uses the Microsoft Speech API (SAPI5) on Windows, which works out of the box. The `sounddevice` package requires PortAudio, which is included as a pre-built wheel for Windows on PyPI — no manual installation needed.

**`[companion]` group on Windows:**
The `pystray` package works natively on Windows 10/11 with the system tray. No additional system packages are required.

**`[ssh]` group on Windows:**
The `asyncssh` package is pure Python and works on Windows without any system-level OpenSSH dependency.

#### Alternative: Using the PowerShell install script

ManusClaw provides an `install.ps1` script for automated Windows installation. See [Using the Install Scripts](#using-the-install-scripts) for details.

---

### WSL2 (Recommended)

WSL2 provides a full Linux environment inside Windows, which eliminates virtually all Windows-specific compatibility issues. For detailed WSL2 setup instructions, see the dedicated [WSL2 Guide](platforms/wsl.md).

#### Step 1: Enable WSL2

Open PowerShell as Administrator and run:

```powershell
wsl --install
```

This installs Ubuntu by default. Restart your computer when prompted.

If you want to install a specific distro:

```powershell
wsl --install -d Ubuntu-22.04
```

#### Step 2: Set up your WSL2 Ubuntu environment

Launch WSL2 from the Start menu or by typing `wsl` in a terminal. Then follow the Ubuntu/Debian instructions from earlier in this guide:

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Python 3.11+ (Ubuntu 22.04 has 3.10, so add deadsnakes PPA)
sudo apt install -y software-properties-common
sudo add-apt-repository -y ppa:deadsnakes/ppa
sudo apt update
sudo apt install -y python3.11 python3.11-venv python3.11-dev python3-pip

# Install system dependencies
sudo apt install -y build-essential libssl-dev libffi-dev python3-dev \
    python3-venv git curl portaudio19-dev libasound2-dev

# Install ManusClaw with all optional dependencies
python3.11 -m pip install --upgrade pip
pip install manusclaw[all-plus]

# Install Playwright browsers
playwright install --with-deps chromium

# Verify
manusclaw --version
```

#### Step 3: Audio access in WSL2 (for `[voice]` group)

Starting with WSL2 on Windows 11 (build 22449+), you can access your Windows audio devices. This is required for the `[voice]` optional group. Ensure your WSL2 has audio forwarding enabled:

```bash
# Check if audio devices are visible in WSL2
python3 -c "import sounddevice; print(sounddevice.query_devices())"
```

If no devices are listed, you may need to update Windows or use the native Windows installation instead for voice features.

#### Step 4: Access Windows files from WSL2

Your Windows C: drive is mounted at `/mnt/c/` in WSL2. You can access your Windows files and vice versa:

```bash
# Navigate to your Windows user directory
cd /mnt/c/Users/YourName/

# WSL2 files are accessible from Windows at:
# \\wsl$\Ubuntu\home\yourname\
```

---

## Docker Installation

Docker is an excellent way to run ManusClaw because it encapsulates all dependencies in a container, ensuring a consistent environment regardless of your host OS. This is the recommended approach for server deployments and for users who want the simplest possible setup.

### Step 1: Install Docker

**Linux (Ubuntu/Debian):**

```bash
# Install Docker using the official script
curl -fsSL https://get.docker.com | sudo sh

# Add your user to the docker group (avoids needing sudo)
sudo usermod -aG docker $USER

# Log out and back in for the group change to take effect
# Or run: newgrp docker
```

**macOS:**

```bash
brew install --cask docker
# Then open Docker from Applications
```

**Windows:**

Download and install [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/). Make sure WSL2 backend is enabled in Docker Desktop settings.

### Step 2: Pull and run the ManusClaw Docker image

The ManusClaw repository includes a Dockerfile and `docker-compose.yml` with **profiles** (new in v5.0.0). You can either use the pre-built image or build from source.

**Option A: Using the pre-built image (when available):**

```bash
docker run -it --rm \
  -e OPENAI_API_KEY="sk-your-key-here" \
  -v $(pwd)/workspace:/app/workspace \
  -v $(pwd)/config:/root/.manusclaw \
  manusclaw/manusclaw:v5.0.0
```

**Option B: Build from source:**

```bash
# Clone the repository
git clone https://github.com/ManusAgents/manusclaw.git
cd manusclaw

# Build the image
docker build -t manusclaw:v5.0.0 .

# Run the container
docker run -it --rm \
  -e OPENAI_API_KEY="sk-your-key-here" \
  -v $(pwd)/workspace:/app/workspace \
  -v $(pwd)/config:/root/.manusclaw \
  manusclaw:v5.0.0
```

### Docker Compose Profiles

Starting with v5.0.0, the `docker-compose.yml` uses **Compose profiles** instead of the deprecated `version: "3.8"` top-level key. Profiles allow you to define multiple service configurations in a single compose file and selectively start them.

The available profiles are:

| Profile | Services Started | Use Case |
|---------|-----------------|----------|
| `server` | `manusclaw-server`, `nginx`, `redis` | Production web server with reverse proxy and caching |
| `cli` | `manusclaw-cli` | Interactive CLI agent in a container |
| `multi-agent` | `manusclaw-server`, `manusclaw-worker-1`, `manusclaw-worker-2`, `redis`, `rabbitmq` | Multi-agent orchestration with task queue |

#### Starting specific profiles

```bash
# Start the server profile
docker compose --profile server up -d

# Start the CLI profile (interactive)
docker compose --profile cli up

# Start the multi-agent profile
docker compose --profile multi-agent up -d

# Start multiple profiles simultaneously
docker compose --profile server --profile multi-agent up -d
```

#### Example: docker-compose.yml with profiles

Here is the structure of the v5.0.0 `docker-compose.yml`. The `version` key is intentionally omitted (per Docker Compose v2+ best practices):

```yaml
# docker-compose.yml — ManusClaw v5.0.0
# No "version" key (Docker Compose v2+ ignores it)

services:
  # ── Server profile ──────────────────────────────────────────
  manusclaw-server:
    build:
      context: .
      dockerfile: Dockerfile
    profiles: ["server", "multi-agent"]
    ports:
      - "8000:8000"
    volumes:
      - ./workspace:/app/workspace
      - ./config:/root/.manusclaw
    env_file:
      - .env
    depends_on:
      redis:
        condition: service_healthy
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    profiles: ["server"]
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - manusclaw-server
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    profiles: ["server", "multi-agent"]
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
    restart: unless-stopped

  # ── Multi-agent profile ─────────────────────────────────────
  manusclaw-worker-1:
    build:
      context: .
      dockerfile: Dockerfile
    profiles: ["multi-agent"]
    command: manusclaw-multi worker --id worker-1
    volumes:
      - ./workspace:/app/workspace
      - ./config:/root/.manusclaw
    env_file:
      - .env
    depends_on:
      redis:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
    restart: unless-stopped

  manusclaw-worker-2:
    build:
      context: .
      dockerfile: Dockerfile
    profiles: ["multi-agent"]
    command: manusclaw-multi worker --id worker-2
    volumes:
      - ./workspace:/app/workspace
      - ./config:/root/.manusclaw
    env_file:
      - .env
    depends_on:
      redis:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
    restart: unless-stopped

  rabbitmq:
    image: rabbitmq:3-management-alpine
    profiles: ["multi-agent"]
    ports:
      - "5672:5672"
      - "15672:15672"
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
    restart: unless-stopped

  # ── CLI profile ─────────────────────────────────────────────
  manusclaw-cli:
    build:
      context: .
      dockerfile: Dockerfile
    profiles: ["cli"]
    stdin_open: true
    tty: true
    volumes:
      - ./workspace:/app/workspace
      - ./config:/root/.manusclaw
    env_file:
      - .env
```

#### Docker Compose profiles explained

**`server` profile:**
- Starts `manusclaw-server`, `nginx`, and `redis`
- `nginx` acts as a reverse proxy with optional SSL termination
- `redis` provides caching and session storage
- Best for: production deployments, always-on access, team use

**`cli` profile:**
- Starts a single interactive `manusclaw-cli` container
- `stdin_open: true` and `tty: true` enable interactive terminal use
- Best for: quick testing, single-user sessions, CI/CD pipelines

**`multi-agent` profile:**
- Starts the server, two worker agents, `redis`, and `rabbitmq`
- Workers communicate via RabbitMQ message queue for task distribution
- `redis` stores shared state and intermediate results
- Best for: parallel task execution, complex multi-step workflows
- You can scale workers by adding more `manusclaw-worker-N` services or using `docker compose up --scale manusclaw-worker-1=5`

### Docker volume mounts

| Volume Mount | Purpose |
|-------------|---------|
| `./workspace:/app/workspace` | Maps your local workspace directory into the container so ManusClaw can read/write your project files |
| `./config:/root/.manusclaw` | Persists your configuration (config.toml, .env, MEMORY.md, USER.md) across container restarts |
| `./skills:/app/skills` | (Optional) Mount custom skills into the container |
| `./nginx/ssl:/etc/nginx/ssl` | (Server profile only) Mount SSL certificates for HTTPS |

> **Important:** Without volume mounts, all data is lost when the container is removed. Always use volume mounts for persistent data.

### Docker and optional dependency groups

To build a Docker image that includes specific optional dependency groups, you have two options:

**Option A: Build with build args**

```bash
docker build \
  --build-arg EXTRAS=all-plus \
  -t manusclaw:v5.0.0-full .
```

The Dockerfile should include:

```dockerfile
ARG EXTRAS=""
RUN pip install -e ".${EXTRAS:+[$EXTRAS]}"
```

**Option B: Use pre-built variant images**

When official images are available, they come in variants:

```bash
docker pull manusclaw/manusclaw:v5.0.0           # Core only
docker pull manusclaw/manusclaw:v5.0.0-voice     # Core + voice
docker pull manusclaw/manusclaw:v5.0.0-full      # All optional deps
```

---

## Termux (Android) Installation

Termux allows you to run a full Linux environment on your Android device. This is useful for running ManusClaw on a phone or tablet, or on an Android TV box as a low-cost always-on server. For a more detailed guide, see the [Termux Platform Guide](platforms/termux.md).

### Step 1: Install Termux

**Important:** Do NOT install Termux from the Google Play Store. The Play Store version is outdated and non-functional due to Google's API level restrictions. Instead, install from F-Droid or get the APK directly from GitHub:

- **F-Droid:** [https://f-droid.org/packages/com.termux/](https://f-droid.org/packages/com.termux/)
- **GitHub:** [https://github.com/termux/termux-app/releases](https://github.com/termux/termux-app/releases)

### Step 2: Update Termux packages

```bash
pkg update && pkg upgrade -y
```

### Step 3: Install required packages

```bash
pkg install -y python python-pip git build-essential binutils

# Optional: for [voice] group (limited on Termux)
pkg install -y portaudio
```

Termux's `python` package typically provides Python 3.11+. Verify:

```bash
python --version
```

### Step 4: Install ManusClaw

```bash
# Core only
pip install manusclaw

# With SSH support (works well on Termux)
pip install manusclaw[ssh]

# With all optional dependencies (some may have limited functionality)
pip install manusclaw[all-plus]
```

> **Note:** Some dependencies may fail to compile on Termux due to missing system libraries. If you encounter build errors, check the [Troubleshooting Guide](troubleshooting.md) for Termux-specific solutions. The `[companion]` group is particularly problematic on Termux since Android does not have a traditional system tray.

### Step 5: Using the Termux setup script

ManusClaw v5.0.0 includes a dedicated Termux setup script (`setup-termux.sh`) that handles the entire installation process, including dependency resolution:

```bash
git clone https://github.com/ManusAgents/manusclaw.git
cd manusclaw
bash setup-termux.sh
```

The `setup-termux.sh` script:
- Detects your Termux environment and Android API level
- Installs all required system packages via `pkg`
- Installs Python 3.11+ if the bundled version is too old
- Installs ManusClaw with compatible optional groups
- Configures recommended environment variables
- Performs basic verification

> **Note:** The script avoids installing groups known to have issues on Termux (e.g., `[companion]`). If you want to force-install all groups, use `bash setup-termux.sh --full`.

### Step 6: Configure API keys

```bash
# Set your API key as an environment variable
echo 'export OPENAI_API_KEY="sk-your-key-here"' >> ~/.bashrc
source ~/.bashrc
```

### Step 7: Verify

```bash
manusclaw --version
```

### Termux limitations

Be aware of these limitations when running ManusClaw on Termux:

- **Playwright browsers will not work** — Termux does not support graphical browser automation. Web search via DuckDuckGo's API still works fine.
- **Voice features are limited** — Porcupine wake-word detection may work but sounddevice has limited audio device support on Android. TTS via pyttsx3 is not available on Termux.
- **Companion tray icon does not work** — Android has no system tray. The `[companion]` group will install but the tray app will not function.
- **Memory constraints** — Android may kill background processes. Use `termux-wake-lock` to prevent this:
  ```bash
  termux-wake-lock
  ```
- **Storage access** — To access your phone's shared storage:
  ```bash
  termux-setup-storage
  ```
  This creates a `~/storage/` directory with symlinks to shared storage locations.
- **SSH works well** — The `[ssh]` group is fully functional on Termux and can be used for remote command execution from your Android device.

---

## Google Colab Installation

Google Colab provides a free (or low-cost) cloud environment with GPU access, making it ideal for running ManusClaw with local models via Ollama. For a complete notebook walkthrough, see the [Colab Platform Guide](platforms/colab.md).

### Step 1: Create a new Colab notebook

Go to [colab.research.google.com](https://colab.research.google.com/) and create a new notebook.

### Step 2: Install ManusClaw in a cell

Add a code cell and run:

```python
# Install ManusClaw v5.0.0
!pip install manusclaw

# Or with specific optional groups
!pip install "manusclaw[ssh,gmail]"

# Verify installation
!manusclaw --version
```

> **Note:** Colab typically runs Python 3.11+, so v5.0.0 requirements are met. The `[voice]` and `[companion]` groups will not function in Colab since there's no audio hardware or system tray.

### Step 3: Set API keys

In a new cell:

```python
import os
os.environ['OPENAI_API_KEY'] = 'sk-your-key-here'
```

> **Security Warning:** Never commit notebooks with API keys to public repositories. Use Colab's secret management (click the 🔑 icon in the left sidebar) to store keys securely:

```python
from google.colab import userdata
os.environ['OPENAI_API_KEY'] = userdata.get('OPENAI_API_KEY')
```

### Step 4: Run ManusClaw in single-shot mode

Colab does not support interactive terminals natively, so use single-shot mode:

```python
!manusclaw "Explain what ManusClaw can do"
```

### Step 5: (Optional) Expose via ngrok for interactive access

If you want to use the interactive shell from Colab, you can run the ManusClaw server and expose it via ngrok:

```python
# Install ngrok
!pip install pyngrok

# Start ManusClaw server in the background
import subprocess
process = subprocess.Popen(['manusclaw-server'], stdout=subprocess.PIPE, stderr=subprocess.PIPE)

# Expose via ngrok
from pyngrok import ngrok
public_url = ngrok.connect(8000)
print(f"ManusClaw server accessible at: {public_url}")
```

### Step 6: (Optional) Using the `[gmail]` group in Colab

The `[gmail]` group is particularly useful in Colab for email automation workflows. After installing:

```python
!pip install "manusclaw[gmail]"

# You'll need to set up OAuth2 credentials
# See the Configuration Guide for Gmail setup
```

---

## VPS / Cloud Server Installation

Running ManusClaw on a VPS or cloud server enables always-on access, team collaboration, and production deployment. The following sections cover popular cloud providers, but the instructions are largely the same for any Linux VPS.

### General VPS Setup (applies to all providers)

Before diving into provider-specific instructions, here's the general workflow that applies to any Linux VPS:

1. **Choose an OS image:** Ubuntu 22.04 LTS or 24.04 LTS are recommended for the best compatibility and community support.
2. **SSH into your server:** `ssh root@your-server-ip` or `ssh user@your-server-ip`
3. **Update the system:** `sudo apt update && sudo apt upgrade -y`
4. **Install Python 3.11+ and ManusClaw** following the Linux instructions above
5. **Configure as a systemd service** (see the [Deployment Guide](deployment.md) for details)
6. **Set up a firewall:** At minimum, open SSH (22) and HTTP/HTTPS (80, 443) if using server mode

#### VPS-specific dependency considerations

For VPS deployments, consider which optional dependency groups you need:

| Use Case | Recommended Groups | Notes |
|----------|-------------------|-------|
| Basic API agent | None (core only) | Minimal footprint, fast startup |
| Web browsing agent | None (core) | Playwright included in core |
| Voice-enabled agent | `[voice]` | Requires audio device or ALSA dummy |
| Remote execution agent | `[ssh]` | Pure Python, no system deps needed |
| Email automation agent | `[gmail]` | Requires OAuth2 credential setup |
| Team chat integration | `[matrix]` | Requires Matrix homeserver |
| Always-on with tray | `[companion]` | Headless VPS: tray won't show, but API works |

### DigitalOcean

DigitalOcean's Droplets are affordable and easy to set up, making them a great choice for running ManusClaw.

#### Step 1: Create a Droplet

1. Log in to [DigitalOcean](https://www.digitalocean.com/)
2. Click "Create" → "Droplets"
3. Choose **Ubuntu 22.04 LTS** or **Ubuntu 24.04 LTS**
4. Choose a plan:
   - **Basic** ($4-6/month): Sufficient for API-based usage
   - **CPU-Optimized** ($21+/month): Recommended if running local models
5. Add your SSH key for secure access
6. Click "Create Droplet"

#### Step 2: SSH into your Droplet

```bash
ssh root@your_droplet_ip
```

#### Step 3: Install ManusClaw

```bash
# Update system
apt update && apt upgrade -y

# Install Python 3.11+
apt install -y python3.11 python3.11-venv python3.11-dev python3-pip \
    build-essential git curl

# Optional: audio libraries for [voice] group
apt install -y portaudio19-dev libasound2-dev

# Install ManusClaw
pip install manusclaw

# Or with specific groups for your use case
pip install manusclaw[ssh,gmail,matrix]

# Install Playwright browsers (optional)
playwright install --with-deps chromium
```

#### Step 4: Configure for persistent operation

See the [Deployment Guide](deployment.md) for setting up systemd, Nginx, and SSL.

---

### AWS EC2

Amazon EC2 provides the most extensive range of instance types, including GPU instances for running local models.

#### Step 1: Launch an EC2 instance

1. Go to the [AWS EC2 Console](https://console.aws.amazon.com/ec2/)
2. Click "Launch Instance"
3. Choose **Ubuntu Server 22.04 LTS** or **Ubuntu Server 24.04 LTS** AMI
4. Choose instance type:
   - **t3.micro** (Free Tier eligible): For API-based usage
   - **g4dn.xlarge** ($0.526/hr): For GPU-accelerated local models
   - **m5.large** ($0.096/hr): Good balance of CPU/RAM for most tasks
5. Configure security group: Allow SSH (port 22) and HTTP/HTTPS (ports 80, 443) if you plan to use the server mode
6. Create or select an SSH key pair
7. Launch the instance

#### Step 2: SSH into your instance

```bash
ssh -i your-key.pem ubuntu@your-instance-public-ip
```

#### Step 3: Install ManusClaw

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Add deadsnakes PPA for Python 3.11+ (Ubuntu 22.04 ships with 3.10)
sudo apt install -y software-properties-common
sudo add-apt-repository -y ppa:deadsnakes/ppa
sudo apt update

# Install Python 3.11
sudo apt install -y python3.11 python3.11-venv python3.11-dev python3-pip \
    build-essential git curl

# Optional: for [voice] group
sudo apt install -y portaudio19-dev libasound2-dev

# Install ManusClaw with desired groups
pip install manusclaw[all-plus]

# Install Playwright browsers
playwright install --with-deps chromium
```

#### Step 4: (Optional) Set up GPU drivers for local models

If you're using a GPU instance, install NVIDIA drivers and CUDA:

```bash
# Install NVIDIA drivers
sudo apt install -y nvidia-driver-535

# Verify GPU is accessible
nvidia-smi

# Install Ollama with GPU support
curl -fsSL https://ollama.com/install.sh | sh
```

---

### Google Cloud Platform

GCP's Compute Engine offers competitive pricing and the option for custom machine types.

#### Step 1: Create a VM instance

1. Go to [Google Cloud Console](https://console.cloud.google.com/compute)
2. Click "Create Instance"
3. Choose **Ubuntu 22.04 LTS** or **Ubuntu 24.04 LTS**
4. Choose machine type:
   - **e2-micro** (Free Tier): For basic API-based usage
   - **n1-standard-4**: For heavier workloads
   - **n1-standard-4 + T4 GPU**: For local model inference
5. Under "Firewall", check "Allow HTTP traffic" and "Allow HTTPS traffic" if using server mode
6. Click "Create"

#### Step 2: SSH into your VM

You can SSH directly from the browser using the "SSH" button in the Console, or use the gcloud CLI:

```bash
gcloud compute ssh --zone your-zone your-instance-name
```

#### Step 3: Install ManusClaw

Follow the same Linux installation instructions as above. For GCP, the `gmail` group works especially well since you can use Google Cloud service accounts for authentication.

---

### Microsoft Azure

Azure provides VMs with good enterprise integration and hybrid cloud support.

#### Step 1: Create a Virtual Machine

1. Go to [Azure Portal](https://portal.azure.com/)
2. Click "Create a resource" → "Virtual machine"
3. Choose **Ubuntu Server 22.04 LTS** or **Ubuntu Server 24.04 LTS**
4. Choose VM size:
   - **B1s** (Free Tier eligible): For basic API usage
   - **D2s_v3**: For standard workloads
   - **NC4as_T4_v3**: For GPU workloads
5. Configure inbound port rules: Allow SSH (22), HTTP (80), HTTPS (443)
6. Create the VM

#### Step 2: SSH into your VM

```bash
ssh -i your-key.pem azureuser@your-vm-public-ip
```

#### Step 3: Install ManusClaw

Follow the same Linux installation instructions as above.

---

## Verification

After installing ManusClaw on any platform, run these verification steps to ensure everything is working correctly.

### Check version

```bash
manusclaw --version
# Expected output: ManusClaw v5.0.0
```

### Check available commands

ManusClaw v5.0.0 installs five entry points. Verify they are all available:

```bash
manusclaw --help           # Main interactive agent
manusclaw-server --help    # Server mode (FastAPI/uvicorn)
manusclaw-cron --help      # Cron scheduler
manusclaw-multi --help     # Multi-agent orchestrator
manusclaw-tray --help      # System tray companion (only if [companion] installed)
```

### Test with a simple query

```bash
# Single-shot mode (requires an API key to be configured)
manusclaw "What is 2 + 2?"
```

If you see a response, ManusClaw is fully operational. If you get an error about API keys, that's expected — you need to configure at least one LLM provider. See the [Configuration Guide](configuration.md).

### Verify Python dependencies

```bash
python3 -c "import manusclaw; print('ManusClaw v5.0.0 imports successfully')"
```

### Verify Playwright (if installed)

```bash
python3 -c "from playwright.sync_api import sync_playwright; print('Playwright is working')"
```

### Verify optional group dependencies

Run these checks for each optional group you installed:

```bash
# [voice] group
python3 -c "import pvporcupine, sounddevice, speech_recognition, pyttsx3; print('Voice dependencies OK')"

# [ssh] group
python3 -c "import asyncssh; print(f'asyncssh {asyncssh.__version__} OK')"

# [gmail] group
python3 -c "from googleapiclient.discovery import build; print('Gmail client OK')"

# [matrix] group
python3 -c "import nio; print(f'matrix-nio {nio.__version__} OK')"

# [companion] group
python3 -c "
import sys
if sys.platform == 'darwin':
    import rumps; print('rumps (macOS companion) OK')
else:
    import pystray; print('pystray (Linux/Windows companion) OK')
"
```

### Comprehensive health check

ManusClaw v5.0.0 includes a built-in health check command:

```bash
manusclaw-doctor
```

This command checks:
- Python version (must be 3.11+)
- All installed dependencies
- Optional group availability
- Playwright browser installation
- Configuration file validity
- API key presence (without revealing values)
- Network connectivity to common API endpoints

---

## Post-Installation Setup

After verifying that ManusClaw is installed, you need to configure it before first use.

### 1. Initialize the configuration

```bash
# Launch ManusClaw for the first time — it will create default config files
manusclaw

# Or manually create the config directory
mkdir -p ~/.manusclaw
```

### 2. Set up your API keys

Edit `~/.manusclaw/.env` or set environment variables:

```bash
# Option A: Environment variables (session-only)
export OPENAI_API_KEY="sk-your-key-here"
export ANTHROPIC_API_KEY="sk-ant-your-key-here"

# Option B: Add to your shell profile (persistent)
echo 'export OPENAI_API_KEY="sk-your-key-here"' >> ~/.bashrc
source ~/.bashrc

# Option C: Use the .env file
nano ~/.manusclaw/.env
```

### 3. Configure your preferred provider

Edit `~/.manusclaw/config.toml`:

```toml
[llm]
provider = "openai"
model = "gpt-4o"
```

See the [Configuration Guide](configuration.md) for complete details on all configuration options.

### 4. Configure optional features (v5.0.0)

If you installed optional dependency groups, add their configuration to `config.toml`:

```toml
# Voice configuration (if [voice] group installed)
[voice]
wake_word = "hey manusclaw"
tts_engine = "elevenlabs"       # or "pyttsx3" for offline
elevenlabs_voice = "Rachel"
porcupine_access_key = "${PORCUPINE_ACCESS_KEY}"

# SSH configuration (if [ssh] group installed)
[ssh]
known_hosts_file = "~/.ssh/known_hosts"
default_user = "root"

# Gmail configuration (if [gmail] group installed)
[gmail]
credentials_file = "~/.manusclaw/gmail_credentials.json"
token_file = "~/.manusclaw/gmail_token.json"

# Matrix configuration (if [matrix] group installed)
[matrix]
homeserver = "https://matrix.org"
user_id = "@youruser:matrix.org"
device_id = "manusclaw-v5"

# Companion configuration (if [companion] group installed)
[companion]
autostart = true
notifications = true
```

### 5. Set up the workspace

```bash
# Create your default workspace
mkdir -p ~/workspace

# Or configure a custom workspace location in config.toml
# [workspace]
# path = "/path/to/your/project"
```

### 6. Start using ManusClaw

```bash
# Interactive mode
manusclaw

# Or with a specific entry point
manusclaw-server    # Web server mode
manusclaw-tray      # System tray companion (if [companion] installed)
manusclaw-multi     # Multi-agent orchestrator
```

You're now ready to use ManusClaw! Head to the [Usage Guide](usage.md) to learn about all the features and commands available to you.

---

## Using the Install Scripts

ManusClaw v5.0.0 provides automated install scripts for convenience. These scripts handle prerequisite installation, Python setup, and ManusClaw installation in a single command.

### Linux / macOS — install.sh

The `install.sh` script is the recommended way to install ManusClaw on Linux and macOS with a single command.

```bash
# Download and run the install script
curl -fsSL https://raw.githubusercontent.com/ManusAgents/manusclaw/v5.0.0/install.sh | bash

# Or clone and run locally
git clone https://github.com/ManusAgents/manusclaw.git
cd manusclaw
git checkout v5.0.0
bash install.sh
```

The `install.sh` script:
- Detects your OS and package manager (apt, dnf, pacman, brew)
- Installs Python 3.11+ if not already present
- Installs pip and other system dependencies (including portaudio for voice)
- Installs ManusClaw via pip (with optional `[all-plus]` groups if you pass `--full`)
- Optionally installs Playwright browsers
- Performs basic verification (`manusclaw --version`)
- Prints a summary of what was installed and what to configure next

**Script options:**

```bash
# Install core only (default)
bash install.sh

# Install with all optional dependencies
bash install.sh --full

# Skip Playwright browser installation
bash install.sh --no-playwright

# Install to a specific Python version
bash install.sh --python 3.12

# Non-interactive mode (for CI/CD)
bash install.sh --yes
```

---

### Windows — install.ps1

The `install.ps1` script is the PowerShell equivalent for Windows users.

```powershell
# Clone the repository
git clone https://github.com/ManusAgents/manusclaw.git
cd manusclaw
git checkout v5.0.0

# Run the PowerShell install script
powershell -ExecutionPolicy Bypass -File install.ps1
```

The `install.ps1` script:
- Checks for Python 3.11+ and offers to install it via the official installer if missing
- Upgrades pip to the latest version
- Installs ManusClaw and dependencies
- Installs Playwright browsers
- Adds manusclaw to PATH if needed
- Installs optional dependency groups based on user prompts

**Script options:**

```powershell
# Install with all optional dependencies
.\install.ps1 -Full

# Skip Playwright
.\install.ps1 -NoPlaywright

# Non-interactive mode
.\install.ps1 -Unattended
```

---

### Termux — setup-termux.sh

The `setup-termux.sh` script is specifically designed for the Termux environment on Android. See also [Termux Installation](#termux-android-installation).

```bash
git clone https://github.com/ManusAgents/manusclaw.git
cd manusclaw
git checkout v5.0.0
bash setup-termux.sh
```

The `setup-termux.sh` script:
- Detects your Termux environment and Android API level
- Installs all required system packages via `pkg`
- Installs Python 3.11+ if the bundled version is too old
- Installs ManusClaw with compatible optional groups (avoids known-broken ones)
- Configures recommended environment variables for Termux
- Performs basic verification
- Sets up `termux-wake-lock` by default

**Script options:**

```bash
# Install with all groups (some may not work)
bash setup-termux.sh --full

# Install specific groups only
bash setup-termux.sh --groups ssh,gmail

# Skip all optional groups (core only)
bash setup-termux.sh --minimal
```

---

## Using pyenv (Alternative Python Installation)

If your system Python is outdated and you can't or don't want to modify it, `pyenv` is an excellent tool for managing multiple Python versions without needing sudo access. This is particularly useful on shared servers, CI/CD environments, or older Linux distributions.

```bash
# Install pyenv
curl https://pyenv.run | bash

# Add pyenv to your shell
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
source ~/.bashrc

# Install Python 3.11 (minimum for v5.0.0)
pyenv install 3.11.9

# Or install Python 3.12 (recommended)
pyenv install 3.12.4

# Set as global default
pyenv global 3.12.4

# Verify
python --version
# Should show Python 3.12.4

# Now install ManusClaw
pip install manusclaw
# Or: pip install manusclaw[all-plus]
```

For pyenv on macOS with Apple Silicon, you may need to set additional compiler flags:

```bash
# Apple Silicon: ensure Homebrew OpenSSL is found
export LDFLAGS="-L/opt/homebrew/opt/openssl@3/lib"
export CPPFLAGS="-I/opt/homebrew/opt/openssl@3/include"
export PKG_CONFIG_PATH="/opt/homebrew/opt/openssl@3/lib/pkgconfig"
pyenv install 3.12.4
```

---

## Virtual Environments

It is strongly recommended to use a virtual environment when installing ManusClaw from source or when developing. This isolates ManusClaw's dependencies from your system Python.

### Using venv (built-in)

```bash
# Create a virtual environment
python3 -m venv ~/manusclaw-env

# Activate it
source ~/manusclaw-env/bin/activate    # Linux/macOS
# Or: ~/manusclaw-env\Scripts\activate  # Windows

# Install ManusClaw inside the venv
pip install manusclaw[all-plus]

# When done, deactivate
deactivate
```

### Using virtualenvwrapper (optional)

```bash
# Install virtualenvwrapper
pip install virtualenvwrapper

# Add to shell profile
echo 'export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3' >> ~/.bashrc
echo 'source ~/.local/bin/virtualenvwrapper.sh' >> ~/.bashrc
source ~/.bashrc

# Create a virtual environment
mkvirtualenv manusclaw-env

# Install ManusClaw
pip install manusclaw[all-plus]

# Switch to the environment later
workon manusclaw-env
```

### Using conda (optional)

```bash
# Create a conda environment with Python 3.11+
conda create -n manusclaw python=3.11 -y
conda activate manusclaw

# Install ManusClaw
pip install manusclaw[all-plus]
```

> **Note:** ManusClaw is not distributed via conda-forge. Install via pip even within a conda environment.

---

## Upgrading from v4.0.0

If you are upgrading from ManusClaw v4.0.0 to v5.0.0, follow these steps:

### Step 1: Backup your configuration

```bash
cp -r ~/.manusclaw ~/.manusclaw.backup-v4
```

### Step 2: Upgrade the package

```bash
# PyPI upgrade (core)
pip install --upgrade manusclaw

# Or with all new optional dependencies
pip install --upgrade manusclaw[all-plus]
```

### Step 3: Review configuration changes

v5.0.0 introduces new configuration sections. Compare your existing `config.toml` with the new defaults:

```bash
# See the new default config
manusclaw config show-defaults > /tmp/default-config.toml
diff ~/.manusclaw/config.toml /tmp/default-config.toml
```

New configuration sections in v5.0.0:
- `[voice]` — wake word, TTS engine, voice selection
- `[ssh]` — remote execution settings
- `[gmail]` — email integration settings
- `[matrix]` — chat integration settings
- `[companion]` — system tray app settings

### Step 4: Run the health check

```bash
manusclaw-doctor
```

This will identify any missing dependencies or configuration issues after the upgrade.

### Step 5: Verify version

```bash
manusclaw --version
# Expected: ManusClaw v5.0.0
```

### Breaking changes from v4.0.0

| Area | Change | Action Required |
|------|--------|----------------|
| Python version | Minimum raised from 3.11 (recommended) to **3.11 (required)** | Upgrade Python if using 3.10 |
| Docker Compose | `version: "3.8"` removed | Use `docker compose` (v2) without version key |
| CLI entry points | Added `manusclaw-tray` (for companion app) | No action needed |
| Configuration | New `[voice]`, `[ssh]`, `[gmail]`, `[matrix]`, `[companion]` sections | Add to config.toml if using those features |
| Dependencies | `asyncssh`, `pvporcupine`, `sounddevice`, `speech-recognition`, `pyttsx3`, `elevenlabs`, `matrix-nio`, `pystray`, `rumps`, `google-api-python-client` are now optional | Install via `[all-plus]` or individual groups |

---

## Next Steps

After completing the installation, proceed to these guides:

1. **[Configuration Guide](configuration.md)** — Set up LLM providers, API keys, and optional features
2. **[Usage Guide](usage.md)** — Learn ManusClaw commands, modes, and workflows
3. **[Features Guide](features.md)** — Explore v5.0.0 features including voice, SSH, Gmail, Matrix, and companion
4. **[Deployment Guide](deployment.md)** — Production deployment with systemd, Nginx, SSL, and Docker
5. **[Troubleshooting Guide](troubleshooting.md)** — Common issues and solutions
