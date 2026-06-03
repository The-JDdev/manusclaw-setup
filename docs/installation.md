# Installation Guide — ManusClaw v4.0.0

This guide covers every platform and method for installing ManusClaw. Each section is self-contained so you can jump directly to your platform. If you encounter any issues, consult the [Troubleshooting Guide](troubleshooting.md) before opening a GitHub issue.

---

## Table of Contents

- [Prerequisites](#prerequisites)
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
- [Termux (Android) Installation](#termux-android-installation)
- [Google Colab Installation](#google-colab-installation)
- [VPS / Cloud Server Installation](#vps--cloud-server-installation)
  - [DigitalOcean](#digitalocean)
  - [AWS EC2](#aws-ec2)
  - [Google Cloud Platform](#google-cloud-platform)
  - [Microsoft Azure](#microsoft-azure)
- [Verification](#verification)
- [Post-Installation Setup](#post-installation-setup)

---

## Prerequisites

Before installing ManusClaw, ensure your system meets these baseline requirements:

| Requirement | Details |
|-------------|---------|
| **Python** | 3.11 or newer (3.12+ recommended) |
| **pip** | Latest version (bundled with Python, but should be updated) |
| **Git** | For cloning the repository if installing from source |
| **Internet** | Required for API-based providers and package downloads |
| **Disk Space** | At least 500 MB free (2 GB+ recommended for Playwright browsers) |
| **RAM** | 512 MB minimum (2 GB+ recommended; more for local models) |

**Why Python 3.11+?** ManusClaw uses modern Python features including the `tomllib` module (added in 3.11), improved error messages, and performance optimizations that are not available in older Python versions. Python 3.11 also includes significant speed improvements (up to 25% faster than 3.10) which directly benefit the agent's responsiveness.

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

**Ubuntu 22.04+ and Debian 12+** ship with Python 3.10+ or 3.11 by default. Check your version:

```bash
python3 --version
```

If you already have Python 3.11+, skip to Step 3.

**Ubuntu 20.04 and Debian 11** ship with Python 3.8 or 3.9, which is too old. You need to install a newer version:

```bash
# Add the deadsnakes PPA (Ubuntu only)
sudo apt install -y software-properties-common
sudo add-apt-repository -y ppa:deadsnakes/ppa
sudo apt update

# Install Python 3.11
sudo apt install -y python3.11 python3.11-venv python3.11-dev

# Set Python 3.11 as the default (optional but recommended)
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 1
```

For **Debian 11**, you may need to build Python from source or use the `pyenv` method described later.

#### Step 3: Install pip and dependencies

```bash
# Install pip
sudo apt install -y python3-pip

# Install other dependencies that ManusClaw or its packages may need
sudo apt install -y build-essential libssl-dev libffi-dev \
    python3-dev python3-venv git curl

# Upgrade pip to the latest version
python3 -m pip install --upgrade pip
```

#### Step 4: Install ManusClaw

You have two options: install from PyPI (recommended) or install from source.

**Option A: Install from PyPI (recommended)**

```bash
pip install manusclaw
```

**Option B: Install from source**

Installing from source gives you access to the latest development changes before they are released on PyPI. This is useful if you want bleeding-edge features or need to contribute code:

```bash
# Clone the repository
git clone https://github.com/The-JDdev/manusclaw.git
cd manusclaw

# Install in editable mode (changes to source are reflected immediately)
pip install -e .

# Or install a specific version/tag
git checkout v4.0.0
pip install -e .
```

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

You should see output like `ManusClaw v4.0.0`. If you see an error, consult the [Troubleshooting Guide](troubleshooting.md).

---

### Fedora / RHEL / CentOS

Fedora typically ships with very recent Python versions, making installation straightforward. RHEL and CentOS may require additional repositories.

#### Step 1: Update your system

```bash
sudo dnf update -y
```

#### Step 2: Install Python 3.11+

**Fedora 37+** ships with Python 3.11+ by default. Check your version:

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
```

#### Step 4: Install ManusClaw

```bash
# Upgrade pip
python3 -m pip install --upgrade pip

# Install ManusClaw
pip install manusclaw
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
```

Arch's Python is typically the latest stable release (3.12+ at the time of writing), so you should be all set. Verify:

```bash
python3 --version
```

#### Step 3: Install ManusClaw

```bash
pip install manusclaw
```

If you prefer to use Arch's AUR (Arch User Repository), check if a `manusclaw` package exists:

```bash
# Using yay
yay -S manusclaw

# Using paru
paru -S manusclaw
```

> **Note:** AUR packages are community-maintained and may not always be up to date. The PyPI method (`pip install`) is the most reliable way to get the latest version.

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
pip3.11 install manusclaw
```

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

Apple Silicon Macs provide excellent performance for running ManusClaw, especially if you use Ollama to run local models on the Neural Engine / GPU.

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
pip3.11 install manusclaw
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

#### Step 6: Verify

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

# Install ManusClaw
pip install manusclaw
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

#### Alternative: Using the PowerShell install script

ManusClaw provides an `install.ps1` script for automated Windows installation:

```powershell
# Clone the repository first
git clone https://github.com/The-JDdev/manusclaw.git
cd manusclaw

# Run the PowerShell install script
powershell -ExecutionPolicy Bypass -File install.ps1
```

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

# Install ManusClaw
python3.11 -m pip install --upgrade pip
pip install manusclaw

# Install Playwright browsers
playwright install --with-deps chromium

# Verify
manusclaw --version
```

#### Step 3: Access Windows files from WSL2

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

The ManusClaw repository includes a Dockerfile and docker-compose.yml. You can either use the pre-built image or build from source.

**Option A: Using the pre-built image (when available):**

```bash
docker run -it --rm \
  -e OPENAI_API_KEY="sk-your-key-here" \
  -v $(pwd)/workspace:/app/workspace \
  -v $(pwd)/config:/root/.manusclaw \
  manusclaw/manusclaw:latest
```

**Option B: Build from source:**

```bash
# Clone the repository
git clone https://github.com/The-JDdev/manusclaw.git
cd manusclaw

# Build the image
docker build -t manusclaw .

# Run the container
docker run -it --rm \
  -e OPENAI_API_KEY="sk-your-key-here" \
  -v $(pwd)/workspace:/app/workspace \
  -v $(pwd)/config:/root/.manusclaw \
  manusclaw
```

### Step 3: Using docker-compose

For more manageable configurations, use docker-compose:

```bash
# Clone the repository
git clone https://github.com/The-JDdev/manusclaw.git
cd manusclaw

# Edit the .env file with your API keys
cp .env.example .env
nano .env

# Start ManusClaw
docker compose up -d

# View logs
docker compose logs -f manusclaw

# Stop ManusClaw
docker compose down
```

The `docker-compose.yml` file handles volume mounts, environment variables, port forwarding, and restart policies automatically.

### Docker volume explanations

| Volume Mount | Purpose |
|-------------|---------|
| `./workspace:/app/workspace` | Maps your local workspace directory into the container so ManusClaw can read/write your project files |
| `./config:/root/.manusclaw` | Persists your configuration (config.toml, .env, MEMORY.md, USER.md) across container restarts |
| `./skills:/app/skills` | (Optional) Mount custom skills into the container |

> **Important:** Without volume mounts, all data is lost when the container is removed. Always use volume mounts for persistent data.

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
```

Termux's `python` package typically provides Python 3.11+. Verify:

```bash
python --version
```

### Step 4: Install ManusClaw

```bash
pip install manusclaw
```

> **Note:** Some dependencies may fail to compile on Termux due to missing system libraries. If you encounter build errors, check the [Troubleshooting Guide](troubleshooting.md) for Termux-specific solutions.

### Step 5: Using the Termux setup script

ManusClaw includes a dedicated Termux setup script that handles the entire installation process, including dependency resolution:

```bash
git clone https://github.com/The-JDdev/manusclaw.git
cd manusclaw
bash setup_termux.sh
```

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
- **Memory constraints** — Android may kill background processes. Use `termux-wake-lock` to prevent this:
  ```bash
  termux-wake-lock
  ```
- **Storage access** — To access your phone's shared storage:
  ```bash
  termux-setup-storage
  ```
  This creates a `~/storage/` directory with symlinks to shared storage locations.

---

## Google Colab Installation

Google Colab provides a free (or low-cost) cloud environment with GPU access, making it ideal for running ManusClaw with local models via Ollama. For a complete notebook walkthrough, see the [Colab Platform Guide](platforms/colab.md).

### Step 1: Create a new Colab notebook

Go to [colab.research.google.com](https://colab.research.google.com/) and create a new notebook.

### Step 2: Install ManusClaw in a cell

Add a code cell and run:

```python
# Install ManusClaw
!pip install manusclaw

# Verify installation
!manusclaw --version
```

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

---

## VPS / Cloud Server Installation

Running ManusClaw on a VPS or cloud server enables always-on access, team collaboration, and production deployment. The following sections cover popular cloud providers, but the instructions are largely the same for any Linux VPS.

### General VPS Setup (applies to all providers)

Before diving into provider-specific instructions, here's the general workflow that applies to any Linux VPS:

1. **Choose an OS image:** Ubuntu 22.04 LTS or 24.04 LTS are recommended for the best compatibility and community support.
2. **SSH into your server:** `ssh root@your-server-ip` or `ssh user@your-server-ip`
3. **Update the system:** `sudo apt update && sudo apt upgrade -y`
4. **Install Python and ManusClaw** following the Linux instructions above
5. **Configure as a systemd service** (see the [Deployment Guide](deployment.md) for details)

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

# Install ManusClaw
pip install manusclaw

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
3. Choose **Ubuntu Server 22.04 LTS** AMI
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

# Install ManusClaw
pip install manusclaw

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

Follow the same Linux installation instructions as above.

---

### Microsoft Azure

Azure provides VMs with good enterprise integration and hybrid cloud support.

#### Step 1: Create a Virtual Machine

1. Go to [Azure Portal](https://portal.azure.com/)
2. Click "Create a resource" → "Virtual machine"
3. Choose **Ubuntu Server 22.04 LTS**
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

After installing ManusClaw on any platform, run these verification steps to ensure everything is working correctly:

### Check version

```bash
manusclaw --version
# Expected output: ManusClaw v4.0.0
```

### Check available commands

ManusClaw installs four entry points. Verify they are all available:

```bash
manusclaw --help           # Main interactive agent
manusclaw-server --help    # Server mode (FastAPI/uvicorn)
manusclaw-cron --help      # Cron scheduler
manusclaw-multi --help     # Multi-agent orchestrator
```

### Test with a simple query

```bash
# Single-shot mode (requires an API key to be configured)
manusclaw "What is 2 + 2?"
```

If you see a response, ManusClaw is fully operational. If you get an error about API keys, that's expected — you need to configure at least one LLM provider. See the [Configuration Guide](configuration.md).

### Verify Python dependencies

```bash
python3 -c "import manusclaw; print('ManusClaw imports successfully')"
```

### Verify Playwright (if installed)

```bash
python3 -c "from playwright.sync_api import sync_playwright; print('Playwright is working')"
```

---

## Post-Installation Setup

After verifying that ManusClaw is installed, you need to configure it before first use:

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

### 4. Set up the workspace

```bash
# Create your default workspace
mkdir -p ~/workspace

# Or configure a custom workspace location in config.toml
# [workspace]
# path = "/path/to/your/project"
```

### 5. Start using ManusClaw

```bash
manusclaw
```

You're now ready to use ManusClaw! Head to the [Usage Guide](usage.md) to learn about all the features and commands available to you.

---

## Using the Install Scripts

ManusClaw provides automated install scripts for convenience. These scripts handle prerequisite installation, Python setup, and ManusClaw installation in a single command.

### Linux / macOS (install.sh)

```bash
# Download and run the install script
curl -fsSL https://raw.githubusercontent.com/The-JDdev/manusclaw/main/install.sh | bash

# Or clone and run locally
git clone https://github.com/The-JDdev/manusclaw.git
cd manusclaw
bash install.sh
```

The `install.sh` script:
- Detects your OS and package manager
- Installs Python 3.11+ if not already present
- Installs pip and other dependencies
- Installs ManusClaw via pip
- Optionally installs Playwright browsers
- Performs basic verification

### Windows (install.ps1)

```powershell
# Clone the repository
git clone https://github.com/The-JDdev/manusclaw.git
cd manusclaw

# Run the PowerShell install script
powershell -ExecutionPolicy Bypass -File install.ps1
```

The `install.ps1` script:
- Checks for Python 3.11+ and offers to install it if missing
- Upgrades pip
- Installs ManusClaw and dependencies
- Installs Playwright browsers
- Adds manusclaw to PATH if needed

---

## Using pyenv (Alternative Python Installation)

If your system Python is outdated and you can't or don't want to modify it, `pyenv` is an excellent tool for managing multiple Python versions without needing sudo access:

```bash
# Install pyenv
curl https://pyenv.run | bash

# Add pyenv to your shell
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
source ~/.bashrc

# Install Python 3.11
pyenv install 3.11.9

# Set as global default
pyenv global 3.11.9

# Verify
python --version
# Should show Python 3.11.9

# Now install ManusClaw
pip install manusclaw
```

This approach is especially useful on shared servers or CI/CD environments where you can't modify the system Python.
