# WSL2 Guide — ManusClaw v5.1.0

Windows Subsystem for Linux version 2 (WSL2) provides a full Linux kernel running inside Windows, making it the best way to run ManusClaw on a Windows machine. WSL2 eliminates virtually all Windows-specific compatibility issues while giving you seamless access to your Windows files and tools. This guide covers everything from installing WSL2 to optimizing it for ManusClaw v5.1.0.

---

## Table of Contents

- [Why WSL2 for ManusClaw?](#why-wsl2-for-manusclaw)
- [WSL2 System Requirements](#wsl2-system-requirements)
- [Installing WSL2](#installing-wsl2)
- [Choosing a Linux Distribution](#choosing-a-linux-distribution)
- [Post-Installation WSL2 Setup](#post-installation-wsl2-setup)
- [Installing ManusClaw on WSL2](#installing-manusclaw-on-wsl2)
- [v5.1 Features on WSL2](#v51-features-on-wsl2)
- [File System and Storage](#file-system-and-storage)
- [GPU Support for Local Models](#gpu-support-for-local-models)
- [GUI Support (WSLg)](#gui-support-wslg)
- [Networking and Port Forwarding](#networking-and-port-forwarding)
- [Integrating with Windows Tools](#integrating-with-windows-tools)
- [Performance Optimization](#performance-optimization)
- [Troubleshooting WSL2 Issues](#troubleshooting-wsl2-issues)
- [Useful WSL2 Commands](#useful-wsl2-commands)

---

## Why WSL2 for ManusClaw?

WSL2 is the recommended way to run ManusClaw on Windows for several important reasons:

1. **Full Linux compatibility** — ManusClaw is developed and tested on Linux. WSL2 gives you the real Linux kernel, so everything works as expected.
2. **No Python version headaches** — Windows Python installations often have quirks. WSL2 uses standard Linux Python.
3. **Docker support** — WSL2 integrates with Docker Desktop for container-based deployments and sandbox backends.
4. **Voice support** — While limited compared to native Linux, WSL2 can access Windows audio devices through WSLg and PulseAudio.
5. **Playwright support** — Playwright browsers work in WSL2 with WSLg GUI support.
6. **v5.1 enterprise features** — All enterprise features (Vault, observability, parallel executor) work in WSL2.

---

## WSL2 System Requirements

| Requirement | Details |
|------------|---------|
| **Windows version** | Windows 10 version 2004+ (Build 19041+) or Windows 11 |
| **CPU** | 64-bit processor with Second Level Address Translation (SLAT) |
| **RAM** | 4 GB minimum (8 GB+ recommended) |
| **Virtualization** | Hardware virtualization must be enabled in BIOS |
| **Disk space** | At least 5 GB free for WSL2 + ManusClaw |

### Checking your Windows version

Open PowerShell and run:

```powershell
winver
```

Look for "Version" and "OS Build" — you need Version 2004 or higher (Build 19041+).

### Checking virtualization

Open Task Manager → Performance → CPU → "Virtualization: Enabled"

If it's disabled, enable it in your BIOS/UEFI settings (usually under "CPU Configuration" or "Advanced").

---

## Installing WSL2

### Quick Install (Windows 11 and Windows 10 Build 20262+)

Open PowerShell as Administrator:

```powershell
wsl --install
```

This installs WSL2 with Ubuntu as the default distribution. Restart your computer when prompted.

### Manual Install (older Windows 10)

```powershell
# 1. Enable WSL
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 2. Enable Virtual Machine Platform
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 3. Restart
Restart-Computer

# 4. Download and install the WSL2 Linux kernel update
# https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi

# 5. Set WSL2 as default
wsl --set-default-version 2
```

### Verify WSL2 installation

```powershell
wsl --list --verbose
# Should show:
#   NAME      STATE     VERSION
# * Ubuntu    Running   2
```

If VERSION shows 1, convert it:

```powershell
wsl --set-version Ubuntu 2
```

---

## Choosing a Linux Distribution

| Distribution | Recommended | Notes |
|-------------|-------------|-------|
| **Ubuntu 22.04/24.04** | ✅ Best choice | Most tested, best documentation |
| **Debian** | ✅ Good | Stable, lighter than Ubuntu |
| **Arch** | ⚠️ Advanced | Rolling release, requires more setup |

Install a distribution:

```powershell
# List available distributions
wsl --list --online

# Install Ubuntu
wsl --install -d Ubuntu-24.04

# Set default distribution
wsl --set-default Ubuntu-24.04
```

---

## Post-Installation WSL2 Setup

After installing WSL2 and your Linux distribution, run these commands inside WSL2:

```bash
# Update all packages
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y build-essential git curl wget python3 python3-pip python3-venv

# Install dependencies for ManusClaw
sudo apt install -y portaudio19-dev libffi-dev libssl-dev libcairo2-dev \
  libjpeg-dev libpng-dev zlib1g-dev

# Install v5.1 enterprise dependencies (optional)
sudo apt install -y libgpgme-dev libdevmapper-dev

# Create a workspace
mkdir -p ~/workspace

# Configure git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

---

## Installing ManusClaw on WSL2

### Step 1: Create a virtual environment

```bash
python3 -m venv ~/manusclaw-env
source ~/manusclaw-env/bin/activate
```

Add the activation to your shell profile for convenience:

```bash
echo 'source ~/manusclaw-env/bin/activate' >> ~/.bashrc
```

### Step 2: Install ManusClaw

```bash
# Full installation with all optional dependencies
pip install "manusclaw[all]"

# Or install specific groups
pip install "manusclaw[voice,enterprise,cloud]"
```

### Step 3: Install Playwright browsers (for web browsing)

```bash
playwright install chromium
playwright install-deps chromium
```

### Step 4: Configure API keys

```bash
mkdir -p ~/.manusclaw
cat > ~/.manusclaw/.env << 'EOF'
OPENAI_API_KEY=sk-proj-your-key-here
ANTHROPIC_API_KEY=sk-ant-your-key-here
EOF
chmod 600 ~/.manusclaw/.env
```

### Step 5: Verify

```bash
manusclaw --version
# manusclaw 5.1.0

manusclaw --validate-config
# Config is valid

manusclaw "Hello, are you working?"
```

---

## v5.1 Features on WSL2

### Feature Availability

| Feature | Available | Notes |
|---------|-----------|-------|
| Single-shot mode | ✅ | Full support |
| Server mode | ✅ | Accessible from Windows |
| All LLM providers | ✅ | Including Ollama with GPU |
| Voice (wake/talk) | ⚠️ | Requires PulseAudio/WSLg |
| Playwright | ✅ | With WSLg |
| SSH Gateway | ✅ | Full support |
| Channels | ✅ | All adapters work |
| Canvas (WebChat) | ✅ | Via browser on Windows |
| Security / RBAC | ✅ | Full support |
| Hooks | ✅ | Shell + Python + webhook |
| Context management | ✅ | Full support |
| Conversation management | ✅ | Full support |
| Observability | ✅ | Full support including Prometheus |
| Secrets (Vault) | ✅ | Can run Vault in Docker |
| File store | ✅ | Local + S3 + GCS + Azure |
| Git providers | ✅ | Full support |
| Parallel executor | ✅ | Threaded + process + Ray |
| Migrations | ✅ | Full support |
| Integrations | ✅ | Full support |
| Docker sandbox | ✅ | With Docker Desktop |

### Setting Up Vault on WSL2

Run HashiCorp Vault using Docker Desktop:

```bash
# Start Vault container
docker run -d \
  --name vault \
  -p 8200:8200 \
  -e VAULT_DEV_ROOT_TOKEN_ID=dev-only-token \
  -e VAULT_DEV_LISTEN_ADDRESS=0.0.0.0:8200 \
  --cap-add IPC_LOCK \
  hashicorp/vault:1.15

# Configure ManusClaw to use Vault
export VAULT_ADDR=http://localhost:8200
export VAULT_TOKEN=dev-only-token

# Test Vault connection
vault status
```

### Setting Up Observability on WSL2

```bash
# Install observability dependencies
pip install "manusclaw[observability]"

# Start OpenTelemetry Collector via Docker
docker run -d \
  --name otel-collector \
  -p 4317:4317 \
  -p 4318:4318 \
  -v ./otel-config.yaml:/etc/otelcol-contrib/config.yaml \
  otel/opentelemetry-collector-contrib:0.96.0

# Start Prometheus via Docker
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v ./prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus:v2.50.0

# Enable observability in config
manusclaw config set observability.enabled true
manusclaw config set observability.telemetry.enabled true
manusclaw config set observability.metrics.enabled true
```

### Setting Up Parallel Executor on WSL2

```bash
# Enable parallel executor
manusclaw config set parallel_executor.enabled true
manusclaw config set parallel_executor.mode threaded
manusclaw config set parallel_executor.workers.max_workers 4

# For Ray (distributed execution)
pip install "manusclaw[parallel]"
ray start --head
manusclaw config set parallel_executor.mode ray
```

---

## File System and Storage

### Where to store ManusClaw files

| Location | Path | Performance | Notes |
|----------|------|-------------|-------|
| **Linux filesystem** | `~/` | ⚡ Fast | Recommended for all ManusClaw files |
| **Windows filesystem** | `/mnt/c/` | 🐌 Slow | Avoid for active work |

**Important:** WSL2 has significantly faster I/O on the Linux filesystem (`~/`) compared to the Windows filesystem (`/mnt/c/`). Always store your ManusClaw workspace and configuration on the Linux filesystem.

```bash
# ✅ Recommended: Linux filesystem
mkdir -p ~/workspace
# ManusClaw config at ~/.manusclaw/

# ❌ Avoid: Windows filesystem
# /mnt/c/Users/username/workspace  — Much slower I/O
```

### Accessing Windows files from WSL2

```bash
# Windows C: drive
ls /mnt/c/

# Your Windows user directory
ls /mnt/c/Users/YourUsername/

# Read a Windows file
manusclaw "Summarize this document" < /mnt/c/Users/YourUsername/Documents/report.txt
```

### Accessing WSL2 files from Windows

In Windows Explorer, navigate to:

```
\\wsl$\Ubuntu\home\yourusername\
```

Or in the WSL2 terminal:

```bash
explorer.exe .
```

---

## GPU Support for Local Models

WSL2 supports GPU passthrough for NVIDIA GPUs, enabling Ollama with hardware acceleration.

### Check GPU support

```bash
# Install NVIDIA CUDA on WSL2
# Follow: https://docs.nvidia.com/cuda/wsl-user-guide/index.html

# Verify GPU access
nvidia-smi
```

### Using Ollama with GPU

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Start Ollama
ollama serve &

# Pull and run a model
ollama pull llama3
manusclaw --provider ollama --model llama3 "Hello from WSL2 with GPU!"
```

### GPU memory allocation

For WSL2, configure GPU memory in `.wslconfig` (on the Windows side):

```ini
# %USERPROFILE%\.wslconfig
[wsl2]
gpu=true
```

---

## GUI Support (WSLg)

WSLg (Windows Subsystem for Linux GUI) allows Linux GUI applications to run on Windows. This enables:

- **Playwright browsers** — Chromium can open windows for web browsing
- **Desktop companion** — `manusclaw-desktop` GUI works
- **Voice features** — Audio through WSLg's PulseAudio integration

### Checking WSLg support

```bash
# Test WSLg
echo $DISPLAY
# Should show something like :0

# Test GUI
xclock  # Should open a clock window
```

### Audio through WSLg

```bash
# Check PulseAudio
pactl info

# Test audio
paplay /usr/share/sounds/alsa/Front_Center.wav

# For voice features, install PyAudio
pip install pyaudio
```

---

## Networking and Port Forwarding

### How WSL2 networking works

WSL2 uses a virtual network adapter. By default, WSL2 ports are automatically forwarded to Windows, so you can access ManusClaw server from Windows:

```bash
# Start ManusClaw server in WSL2
manusclaw-server --host 0.0.0.0 --port 8765

# Access from Windows browser:
# http://localhost:8765
```

### Finding the WSL2 IP address

```bash
# Get WSL2 IP
hostname -I

# Get Windows host IP (for accessing Windows services from WSL2)
cat /etc/resolv.conf | grep nameserver | awk '{print $2}'
```

### Port forwarding issues

If port forwarding doesn't work automatically:

```powershell
# From Windows PowerShell (as Administrator)
netsh interface portproxy add v4tov4 listenport=8765 listenaddress=0.0.0.0 connectport=8765 connectaddress=$(wsl hostname -I)
```

### SSH Gateway on WSL2

```bash
# Start ManusClaw SSH gateway
export MANUSCLAW_SSH_ENABLED=true
export MANUSCLAW_SSH_PORT=2222
manusclaw-ssh start

# Connect from Windows
ssh -p 2222 admin@localhost

# Connect from another machine on the network
ssh -p 2222 admin@windows-ip
```

---

## Integrating with Windows Tools

### Using Windows programs from WSL2

```bash
# Open a file in Windows
explorer.exe .

# Use Windows browser
# WSLg or wslu required
xdg-open https://example.com

# Edit files with VS Code
code .
```

### Using Windows Python from WSL2

It's possible but **not recommended**. Use the WSL2 Python instead for better compatibility and performance.

### Docker Desktop integration

Docker Desktop for Windows integrates with WSL2:

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. Enable WSL2 integration in Docker Desktop settings
3. Use Docker from within WSL2:

```bash
# Verify Docker
docker run hello-world

# Run ManusClaw in Docker
docker run -it --rm \
  -e OPENAI_API_KEY=sk-proj-xxx \
  -v ~/.manusclaw:/root/.manusclaw \
  manusclaw/manusclaw:5.1.0
```

---

## Performance Optimization

### WSL2 Memory Configuration

Create or edit `%USERPROFILE%\.wslconfig` on Windows:

```ini
[wsl2]
memory=8GB
processors=4
swap=4GB
```

Restart WSL2:

```powershell
wsl --shutdown
wsl
```

### File System Optimization

```bash
# Store workspace on Linux filesystem (fastest)
mkdir -p ~/workspace

# Disable Windows Defender scanning for WSL2 paths
# Add exclusions in Windows Security for:
#   \\wsl$\Ubuntu
#   \\wsl.localhost\Ubuntu
```

### Python Optimization

```bash
# Use a virtual environment
python3 -m venv ~/manusclaw-env
source ~/manusclaw-env/bin/activate

# Disable bytecode caching for faster startup
export PYTHONDONTWRITEBYTECODE=1
```

---

## Troubleshooting WSL2 Issues

### Error: WSL2 not starting

```powershell
# Restart WSL
wsl --shutdown
wsl

# Check WSL status
wsl --status

# Re-register the distribution
wsl --unregister Ubuntu
wsl --install -d Ubuntu
```

### Error: `Cannot connect to Docker daemon`

```bash
# Check if Docker Desktop is running
docker info

# Restart Docker Desktop from Windows

# Verify WSL2 integration in Docker Desktop Settings → Resources → WSL Integration
```

### Error: `Playwright browser fails to launch`

```bash
# Install browser dependencies
sudo playwright install-deps chromium

# If WSLg is not available, use headless mode
export PLAYWRIGHT_BROWSERS_PATH=0  # Force headless

# Or set DISPLAY
export DISPLAY=:0
```

### Error: `Audio not working in WSL2`

```bash
# Check WSLg audio
pactl info

# If PulseAudio is not available, install it
sudo apt install -y pulseaudio

# Configure PulseAudio
echo "load-module module-native-protocol-tcp auth-anonymous=1" | sudo tee -a /etc/pulse/default.pa
pulseaudio --start
```

### Error: `Voice features not working`

Voice features in WSL2 depend on audio device access through WSLg or PulseAudio. If they don't work:

```bash
# Test microphone access
python -c "import pyaudio; p=pyaudio.PyAudio(); print(f'Devices: {p.get_device_count()}')"

# If no devices found, use text-based interaction instead
# Voice features work best on native Linux or macOS
```

### Error: v5.1 Vault connection refused from WSL2

```bash
# Check if Vault container is running
docker ps | grep vault

# Verify Vault is accessible
curl http://localhost:8200/v1/sys/health

# If using Docker Desktop, ensure port 8200 is mapped
# Check Docker Desktop → Port Forwarding
```

### Error: v5.1 OTEL collector not receiving data

```bash
# Verify the collector is running
docker ps | grep otel

# Check the OTLP endpoint
echo $OTEL_EXPORTER_OTLP_ENDPOINT

# Test with a manual trace
curl -X POST http://localhost:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{"resourceSpans":[]}'

# Ensure WSL2 can reach the Docker container
curl http://localhost:4317
```

---

## Useful WSL2 Commands

```powershell
# List distributions
wsl --list --verbose

# Start a specific distribution
wsl -d Ubuntu-24.04

# Shutdown all WSL instances
wsl --shutdown

# Convert WSL1 to WSL2
wsl --set-version Ubuntu 2

# Set default distribution
wsl --set-default Ubuntu

# Export a distribution
wsl --export Ubuntu ubuntu-backup.tar

# Import a distribution
wsl --import Ubuntu-Backup D:\WSL\Backup ubuntu-backup.tar

# Update WSL
wsl --update
```

### Inside WSL2

```bash
# Open Windows Explorer at current directory
explorer.exe .

# Open a file with Windows default program
wslview file.pdf

# Copy path to clipboard (Windows)
wslpath -w "$(pwd)/file.txt" | clip.exe

# Run Windows program
notepad.exe /mnt/c/Users/username/file.txt
```
