# WSL2 Guide — ManusClaw v5.0.0

Windows Subsystem for Linux version 2 (WSL2) provides a full Linux kernel running inside Windows, making it the best way to run ManusClaw on a Windows machine. WSL2 eliminates virtually all Windows-specific compatibility issues while giving you seamless access to your Windows files and tools. This guide covers everything from installing WSL2 to optimizing it for ManusClaw.

---

## Table of Contents

- [Why WSL2 for ManusClaw?](#why-wsl2-for-manusclaw)
- [WSL2 System Requirements](#wsl2-system-requirements)
- [Installing WSL2](#installing-wsl2)
- [Choosing a Linux Distribution](#choosing-a-linux-distribution)
- [Post-Installation WSL2 Setup](#post-installation-wsl2-setup)
- [Installing ManusClaw on WSL2](#installing-manusclaw-on-wsl2)
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

1. **Full Linux compatibility** — ManusClaw is developed and tested on Linux. WSL2 gives you the real Linux kernel, not an emulation layer, so everything works as expected.
2. **No Python version headaches** — Windows Python installations often have quirks with path separators, file permissions, and package compilation. WSL2 uses the standard Linux Python.
3. **Shell compatibility** — ManusClaw uses many Unix conventions (pipes, process signals, file permissions). These work natively in WSL2 but can be problematic on Windows.
4. **Playwright works properly** — Playwright's Chromium runs much more reliably on WSL2 than on native Windows, especially with WSLg (Windows 11) providing GUI support.
5. **Ollama GPU support** — WSL2 supports GPU passthrough, so you can run Ollama with NVIDIA GPU acceleration for local models.
6. **Docker integration** — Docker Desktop for Windows uses WSL2 as its backend, providing excellent Docker performance.

---

## WSL2 System Requirements

| Requirement | Details |
|-------------|---------|
| **Windows version** | Windows 10 version 2004+ (Build 19041+) or Windows 11 |
| **Architecture** | x64 or ARM64 |
| **RAM** | 8 GB minimum (16 GB+ recommended for local models) |
| **Virtualization** | Must be enabled in BIOS/UEFI and Windows features |
| **Disk space** | At least 5 GB for WSL2 + Linux distribution |

### Check your Windows version

Open PowerShell and run:

```powershell
winver
```

You need:
- Windows 10: Version 2004 or later (Build 19041 or higher)
- Windows 11: Any version

### Check if virtualization is enabled

Open Task Manager → Performance → CPU. Look for "Virtualization: Enabled."

If virtualization is disabled:
1. Restart your computer and enter BIOS/UEFI setup (usually F2, F12, or Delete during boot)
2. Look for "Intel VT-x", "AMD-V", or "SVM Mode" in the CPU or Security settings
3. Enable it and save changes

---

## Installing WSL2

### One-command install (Windows 10 2004+ and Windows 11)

Open **PowerShell as Administrator** and run:

```powershell
wsl --install
```

This command:
1. Enables the WSL and Virtual Machine Platform features
2. Downloads and installs the latest Linux kernel
3. Installs Ubuntu as the default distribution
4. Sets WSL 2 as the default version

**Restart your computer** after the installation completes.

### Manual install (if the one-command install doesn't work)

Open **PowerShell as Administrator**:

```powershell
# Step 1: Enable WSL
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# Step 2: Enable Virtual Machine Platform
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# Step 3: Restart
Restart-Computer

# Step 4: Download and install the WSL2 Linux kernel update
# Go to: https://aka.ms/wsl2kernel
# Download and run the installer

# Step 5: Set WSL 2 as default
wsl --set-default-version 2
```

### Verify WSL2 is installed

```powershell
wsl --list --verbose
```

You should see output like:

```
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

The VERSION column should show `2` (not `1`). If it shows `1`, convert it:

```powershell
wsl --set-version Ubuntu 2
```

---

## Choosing a Linux Distribution

WSL2 supports multiple Linux distributions. For ManusClaw, we recommend:

| Distribution | Pros | Cons | Recommendation |
|-------------|------|------|---------------|
| **Ubuntu 22.04** | Most popular, best community support, easy to find help | Larger download | ✅ **Best choice** |
| **Ubuntu 24.04** | Newer packages | Some software may not be updated yet | ✅ Good |
| **Debian** | Lightweight, very stable | Older packages | ✅ Good for minimalists |
| **Arch** | Rolling release, always latest | More manual setup, can break | ⚠️ Advanced users only |

### Install a specific distribution

```powershell
# List available distributions
wsl --list --online

# Install Ubuntu 22.04
wsl --install -d Ubuntu-22.04

# Install Debian
wsl --install -d Debian

# Install Arch (from Microsoft Store or AUR)
```

### Set a default distribution

```powershell
wsl --set-default Ubuntu-22.04
```

---

## Post-Installation WSL2 Setup

After installing WSL2 and launching it for the first time, you'll be prompted to create a username and password. This is your Linux user — it has sudo privileges but is not the root user.

### Step 1: Update the system

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 2: Install essential tools

```bash
sudo apt install -y build-essential git curl wget software-properties-common
```

### Step 3: Set up Git (if you haven't already)

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Fix line ending issues (important for WSL!)
git config --global core.autocrlf input
```

**Why `core.autocrlf input`?** Windows uses CRLF line endings (`\r\n`) while Linux uses LF (`\n`). This can cause bizarre issues with scripts and configuration files. Setting `autocrlf input` ensures that git converts CRLF to LF when committing, which prevents most problems.

### Step 4: Configure WSL memory and CPU (optional but recommended)

By default, WSL2 uses up to 50% of your total RAM and all CPU cores. You can adjust this by creating a `.wslconfig` file:

**In Windows (using PowerShell):**

```powershell
notepad "$env:USERPROFILE\.wslconfig"
```

Add the following (adjust values for your system):

```ini
[wsl2]
memory=8GB          # Maximum RAM for WSL2
processors=4        # Number of CPU cores
swap=4GB            # Swap space
localhostForwarding=true

[experimental]
autoMemoryReclaim=gradual  # Return unused memory to Windows
```

After saving, restart WSL2:

```powershell
wsl --shutdown
```

Then relaunch WSL2.

### Step 5: Set up Windows Terminal (recommended)

Windows Terminal provides a much better experience than the default command prompt:

1. Install [Windows Terminal](https://aka.ms/terminal) from the Microsoft Store
2. It automatically detects your WSL2 distributions
3. Configure it as your default terminal:
   - Open Windows Terminal → Settings → Startup → Default profile → Ubuntu

---

## Installing ManusClaw on WSL2

With WSL2 set up, installing ManusClaw follows the standard Linux installation process. ManusClaw v5.0.0 requires **Python 3.11+**.

### Step 1: Install Python 3.11+

Ubuntu 22.04 ships with Python 3.10, which is too old for ManusClaw. Install Python 3.11+:

```bash
# Add the deadsnakes PPA
sudo add-apt-repository -y ppa:deadsnakes/ppa
sudo apt update

# Install Python 3.11 and related packages
sudo apt install -y python3.11 python3.11-venv python3.11-dev python3-pip
```

Ubuntu 24.04 ships with Python 3.12, so you can skip the deadsnakes PPA:

```bash
# Ubuntu 24.04 only
sudo apt install -y python3 python3-venv python3-dev python3-pip
```

### Step 2: Create a virtual environment

```bash
# Create a virtual environment
python3.11 -m venv ~/manusclaw-env

# Activate it
source ~/manusclaw-env/bin/activate

# Add activation to your .bashrc so it's always active
echo 'source ~/manusclaw-env/bin/activate' >> ~/.bashrc
```

### Step 3: Install ManusClaw

```bash
# Upgrade pip
pip install --upgrade pip

# Install ManusClaw
pip install manusclaw
```

### Step 4: Install Playwright browsers

```bash
# Install Chromium with system dependencies
playwright install --with-deps chromium
```

The `--with-deps` flag is important on WSL2 because it installs the graphics libraries that Chromium needs. Without it, Playwright may fail with missing library errors.

### Step 5: Verify installation

```bash
manusclaw --version
manusclaw-server --help
```

### Step 6: Configure API keys

```bash
# Create the config directory
mkdir -p ~/.manusclaw

# Set API keys
echo 'export OPENAI_API_KEY="sk-proj-xxx"' >> ~/.bashrc
source ~/.bashrc
```

### Step 7: Test

```bash
manusclaw "Hello from WSL2!"
```

---

## File System and Storage

Understanding how WSL2 handles files is crucial for a good experience. WSL2 has two file systems with very different performance characteristics.

### The Linux file system (fast)

Files stored in the Linux file system (`/home/username/`) have native Linux performance. This is where you should keep your ManusClaw workspace and all project files.

```bash
# Your home directory - FAST
cd ~
ls

# Create your workspace here
mkdir -p ~/workspace
```

### The Windows file system (slow for Linux operations)

Windows drives are mounted under `/mnt/` in WSL2:

```bash
# Windows C: drive - SLOW for Linux operations
cd /mnt/c/Users/YourName/
ls
```

**Why is the Windows file system slow?** WSL2 accesses Windows files through the 9P protocol, which adds significant overhead for file operations. Git operations (status, diff, log) can be 3-5x slower on `/mnt/c/` compared to the Linux file system.

### Where to put your workspace

**✅ Recommended:** Keep your workspace in the Linux file system:

```bash
# In WSL2
mkdir -p ~/workspace
cd ~/workspace
manusclaw
```

**⚠️ If you must use Windows files:** Use the Windows path for access, but be aware of slower performance:

```bash
# Access a Windows project from WSL2
cd /mnt/c/Users/YourName/projects/my-app
manusclaw
```

### Accessing WSL2 files from Windows

Your WSL2 files are accessible from Windows Explorer at:

```
\\wsl$\Ubuntu\home\username\
\\wsl.localhost\Ubuntu\home\username\
```

You can open this in Explorer by typing `\\wsl$` in the address bar, or from WSL2:

```bash
explorer.exe .
```

This opens the current Linux directory in Windows Explorer. You can also open files with Windows applications:

```bash
# Open a file with the default Windows application
cmd.exe /c start output.html

# Open a file in VS Code (from WSL2)
code .
```

---

## GPU Support for Local Models

WSL2 supports NVIDIA GPU passthrough, which enables running Ollama with GPU acceleration. This is a huge advantage for running local models.

### Prerequisites

1. **NVIDIA GPU** — Only NVIDIA GPUs are officially supported (AMD support is experimental)
2. **Latest NVIDIA driver for Windows** — Install from [nvidia.com/drivers](https://www.nvidia.com/drivers)
3. **WSL2 with GPU support** — Available in Windows 10 21H2+ and Windows 11

> **Important:** Do NOT install NVIDIA drivers inside WSL2. The Windows driver handles everything. WSL2 automatically maps the GPU through.

### Verify GPU access

```bash
# Inside WSL2
nvidia-smi
```

You should see your GPU information. If you see "NVIDIA-SMI has failed," the GPU isn't accessible.

### Install Ollama with GPU support

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# The installer detects WSL2 and sets up GPU support automatically

# Pull and run a model
ollama pull llama3
ollama run llama3
```

### Configure ManusClaw for Ollama on WSL2

```bash
cat > ~/.manusclaw/config.toml << 'EOF'
[llm]
provider = "ollama"
model = "llama3"

[llm.ollama]
base_url = "http://localhost:11434"
model = "llama3"
timeout = 300
EOF
```

### CUDA toolkit (optional, for advanced use)

If you need the CUDA toolkit for compiling GPU-accelerated Python packages:

```bash
# Install CUDA toolkit for WSL2
sudo apt install -y nvidia-cuda-toolkit

# Verify
nvcc --version
```

---

## GUI Support (WSLg)

Windows 11 includes WSLg (Windows Subsystem for Linux GUI), which allows Linux GUI applications to run natively on Windows. This is useful for Playwright browser visualization and other GUI tools.

### Check if WSLg is available

```bash
# Test with a simple GUI app
sudo apt install -y x11-apps
xeyes
```

If a window with eyes that follow your cursor appears, WSLg is working.

### Windows 10 users

WSLg is not officially supported on Windows 10. However, you can set up an X server manually:

1. Install [VcXsrv](https://sourceforge.net/projects/vcxsrv/) on Windows
2. Launch VcXsrv with "Disable access control" checked
3. In WSL2:

```bash
export DISPLAY=$(ip route show default | awk '{print $3}'):0.0
echo 'export DISPLAY=$(ip route show default | awk '"'"'{print $3}'"'"'):0.0' >> ~/.bashrc
```

4. Test:

```bash
sudo apt install -y x11-apps
xeyes
```

---

## Networking and Port Forwarding

WSL2 has its own IP address and network stack, but Windows automatically forwards ports from WSL2 to the host. This means:

- A server running in WSL2 on port 8765 is automatically accessible at `localhost:8765` on Windows
- You can access the ManusClaw v5 server from your Windows browser
- Other devices on your network can access it via your Windows IP address

### Starting the ManusClaw server

```bash
# In WSL2 — v5 default port is 8765
manusclaw-server --host 0.0.0.0 --port 8765
```

### Accessing from Windows

Open your Windows browser and go to:

```
http://localhost:8765/health
```

### Accessing from other devices on your network

1. Find your Windows IP address:

```powershell
# In PowerShell
ipconfig
# Look for your Wi-Fi or Ethernet IPv4 address
```

2. Make sure Windows Firewall allows the port:

```powershell
# In PowerShell (as Administrator)
New-NetFirewallRule -DisplayName "ManusClaw" -Direction Inbound -LocalPort 8765 -Protocol TCP -Action Allow
```

3. Access from another device:

```
http://your-windows-ip:8765/health
```

### Port forwarding issues

Sometimes WSL2's automatic port forwarding doesn't work. If you can't access a WSL2 server from Windows:

```powershell
# In PowerShell (as Administrator)
# Get the WSL2 IP address
wsl hostname -I

# Manually forward the port
netsh interface portproxy add v4tov4 listenport=8765 listenaddress=0.0.0.0 connectport=8765 connectaddress=$(wsl hostname -I)
```

To remove the forwarding:

```powershell
netsh interface portproxy delete v4tov4 listenport=8765 listenaddress=0.0.0.0
```

---

## Integrating with Windows Tools

### Visual Studio Code

VS Code has excellent WSL2 integration through the "Remote - WSL" extension:

1. Install VS Code on Windows from [code.visualstudio.com](https://code.visualstudio.com/)
2. Install the "WSL" extension in VS Code
3. From WSL2, open your project:

```bash
cd ~/workspace
code .
```

VS Code will open with a "WSL: Ubuntu" indicator in the bottom-left corner, meaning it's connected to your WSL2 environment. Extensions like Python, Pylance, and Git will use the WSL2 installation, not Windows.

### Docker Desktop

Docker Desktop for Windows uses WSL2 as its backend. This is the recommended way to use Docker with ManusClaw v5's multi-profile compose files:

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. In Docker Desktop settings → General → "Use the WSL 2 based engine" (should be enabled by default)
3. Under Resources → WSL Integration → Enable integration for your distribution
4. Ensure **Compose V2** is enabled (Docker Desktop enables this by default)

Now you can use Docker from both Windows and WSL2. v5's profile-based compose files work seamlessly:

```bash
# In WSL2
docker ps
docker run manusclaw
```

### Windows Terminal profiles

Add custom profiles to Windows Terminal for quick access to ManusClaw:

1. Open Windows Terminal → Settings → Add a new profile
2. Configure:
   - Name: `ManusClaw`
   - Command line: `wsl -d Ubuntu-22.04 -- bash -c "source ~/manusclaw-env/bin/activate && manusclaw"`
   - Starting directory: `\\wsl$\Ubuntu-22.04\home\username\workspace`
   - Tab title: `ManusClaw`

### Running Windows commands from WSL2

You can call Windows executables from WSL2 by appending `.exe`:

```bash
# Open a file with the default Windows application
cmd.exe /c start output.html

# Open Explorer in the current directory
explorer.exe .

# Run PowerShell
powershell.exe -c "Get-Process"

# Use notepad
notepad.exe config.toml

# Access Windows clipboard
echo "Hello from WSL2" | clip.exe
```

---

## Performance Optimization

### 1. Keep files in the Linux file system

As mentioned earlier, file operations are 3-5x faster in the Linux file system (`/home/`) compared to the Windows file system (`/mnt/c/`). Always keep your ManusClaw workspace and project files in the Linux file system.

### 2. Configure WSL memory limits

Prevent WSL2 from consuming too much RAM:

```ini
# %USERPROFILE%\.wslconfig
[wsl2]
memory=8GB
swap=4GB
```

### 3. Disable Windows Defender scanning for WSL2

Windows Defender can significantly slow down file operations in WSL2. Add exclusions:

1. Open Windows Security → Virus & threat protection → Manage settings
2. Under Exclusions, click "Add or remove exclusions"
3. Add exclusions for:
   - `\\wsl$` (all WSL2 file systems)
   - `\\wsl.localhost` (alternative WSL2 path)
   - The path to your virtual disk: `%LOCALAPPDATA%\Packages\CanonicalGroupLimited.Ubuntu*\LocalState\ext4.vhdx`

### 4. Use Windows Terminal instead of Windows Console

Windows Terminal is significantly faster and more feature-rich than the default console host.

### 5. Enable systemd (optional)

WSL2 now supports systemd, which enables proper service management:

```bash
# Edit /etc/wsl.conf
sudo nano /etc/wsl.conf
```

Add:

```ini
[boot]
systemd=true
```

Restart WSL2:

```powershell
wsl --shutdown
```

Now you can use `systemctl` to manage services:

```bash
systemctl status
sudo systemctl start manusclaw
```

---

## Audio Device Passthrough (v5 Voice Features)

ManusClaw v5.0.0 includes voice features (wake word, talk mode) that require microphone and speaker access. WSL2 has limited audio support, but it can work with configuration.

### Checking audio devices

```bash
# List audio devices visible to WSL2
python3 -c "import pyaudio; p = pyaudio.PyAudio(); [print(f'{i}: {p.get_device_info_by_index(i)[\"name\"]}') for i in range(p.get_device_count())]"
```

### Option 1: Windows 11 with WSLg audio (limited)

Windows 11's WSLg includes basic audio support. If PulseAudio devices are visible, voice features may work out of the box:

```bash
# Install PortAudio development libraries
sudo apt install -y portaudio19-dev
pip install pyaudio

# Test
manusclaw voice talk --start
```

### Option 2: PulseAudio server (recommended for voice)

For reliable audio in WSL2, run a PulseAudio server on Windows and connect WSL2 to it:

1. **Install PulseAudio for Windows:** Download from [freedesktop.org](https://www.freedesktop.org/wiki/Software/PulseAudio/Ports/Windows/Support/)

2. **Configure WSL2 to use the Windows PulseAudio server:**
   ```bash
   echo 'export PULSE_SERVER=tcp:$(cat /etc/resolv.conf | grep nameserver | awk "{print $2}")' >> ~/.bashrc
   source ~/.bashrc
   ```

3. **Install audio dependencies:**
   ```bash
   sudo apt install -y portaudio19-dev pulseaudio-utils
   pip install pyaudio
   ```

4. **Test audio:**
   ```bash
   # Check if PulseAudio is reachable
   pactl info
   ```

### Option 3: USB microphone passthrough

If using a USB microphone:

1. Plug in the USB microphone to your Windows machine
2. In PowerShell, check it's recognized:
   ```powershell
   Get-PnpDevice -Class Audio
   ```
3. WSL2 should see the device through PulseAudio (if configured above)

> **Note:** Voice features are best experienced on native Linux or macOS. WSL2 audio is a work in progress and may have latency or compatibility issues.

---

## Troubleshooting WSL2 Issues

### Error: `WslRegisterDistribution failed with error: 0x8007019e`

**Root Cause:** The WSL feature is not enabled.

**Solution:**

```powershell
# In PowerShell (as Administrator)
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
Restart-Computer
```

### Error: `WslRegisterDistribution failed with error: 0x80370102`

**Root Cause:** Virtualization is not enabled or the Virtual Machine Platform feature is missing.

**Solution:**

1. Enable virtualization in BIOS (see the System Requirements section)
2. Enable the Virtual Machine Platform feature:

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
Restart-Computer
```

### Error: Very slow performance on /mnt/c/

**Root Cause:** This is a known WSL2 limitation. File operations across the Windows/Linux boundary are inherently slower.

**Solution:** Move your project files to the Linux file system:

```bash
# Copy from Windows to Linux
cp -r /mnt/c/Users/YourName/projects/my-app ~/workspace/my-app

# Work from the Linux file system
cd ~/workspace/my-app
```

### Error: `Cannot open display` (GUI apps don't work)

**Root Cause:** WSLg isn't available (Windows 10) or not configured properly.

**Solution for Windows 11:**

```bash
# Verify WSLg is working
echo $WAYLAND_DISPLAY
# Should show: wayland-0

# If empty, try:
export WAYLAND_DISPLAY=wayland-0
export XDG_RUNTIME_DIR=/run/user/$(id -u)
```

**Solution for Windows 10:** See the [GUI Support section](#gui-support-wslg) above for VcXsrv setup.

### Error: DNS resolution fails inside WSL2

**Symptom:** `ping google.com` fails but `ping 8.8.8.8` works.

**Solution:**

```bash
# Back up the existing resolv.conf
sudo cp /etc/resolv.conf /etc/resolv.conf.backup

# Create a custom resolv.conf
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
echo "nameserver 8.8.4.4" | sudo tee -a /etc/resolv.conf

> **Note:** If you're running ManusClaw v5 with Gmail Pub/Sub or webhook endpoints that need inbound connections, ensure DNS is working correctly.

# Prevent WSL from overwriting it
sudo chattr +i /etc/resolv.conf

# Also disable the auto-generated resolv.conf
sudo nano /etc/wsl.conf
```

Add:

```ini
[network]
generateResolvConf = false
```

Restart WSL2:

```powershell
wsl --shutdown
```

### Error: WSL2 consumes too much memory

**Symptom:** Task Manager shows `Vmmem` process using excessive RAM.

**Solution:**

Create or edit `%USERPROFILE%\.wslconfig`:

```ini
[wsl2]
memory=4GB
swap=2GB

[experimental]
autoMemoryReclaim=gradual
```

Restart WSL2:

```powershell
wsl --shutdown
```

### Error: WSL2 won't start after Windows update

**Solution:**

```powershell
# Try a clean shutdown
wsl --shutdown

# Try again
wsl

# If that doesn't work, update WSL
wsl --update

# If still broken, reinstall the distribution
wsl --unregister Ubuntu-22.04
wsl --install -d Ubuntu-22.04
```

> **⚠️ Warning:** `wsl --unregister` deletes all data in that distribution. Back up important files first!

---

## Useful WSL2 Commands

Quick reference for common WSL2 management commands:

```powershell
# Run in PowerShell

# List installed distributions
wsl --list --verbose

# Start a specific distribution
wsl -d Ubuntu-22.04

# Shut down all WSL instances
wsl --shutdown

# Update WSL
wsl --update

# Check WSL status
wsl --status

# Convert a distribution from WSL1 to WSL2
wsl --set-version Ubuntu-22.04 2

# Set the default distribution
wsl --set-default Ubuntu-22.04

# Run a specific command in WSL
wsl -- bash -c "manusclaw --version"

# Check v5 server status
wsl -- bash -c "curl -s http://localhost:8765/health"

# Export a distribution (backup)
wsl --export Ubuntu-22.04 D:\backup\ubuntu-backup.tar

# Import a distribution (restore)
wsl --import Ubuntu-Restored D:\wsl\ D:\backup\ubuntu-backup.tar

# Unregister (delete) a distribution
# ⚠️ THIS DELETES ALL DATA
wsl --unregister Ubuntu-22.04
```

```bash
# Run inside WSL2

# Check WSL version
cat /proc/version

# Check memory usage
free -h

# Check disk usage
df -h

# Access Windows clipboard
echo "Hello" | clip.exe
powershell.exe -c "Get-Clipboard"

# Open Windows Explorer from WSL2
explorer.exe .

# Open a file with the default Windows application
cmd.exe /c start file.html
```
