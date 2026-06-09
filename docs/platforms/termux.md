# Termux (Android) Guide — ManusClaw v5.0.0

Running ManusClaw on Android via Termux transforms your phone or tablet into a portable AI agent. This guide covers everything you need to know about installing, configuring, and using ManusClaw on Android, including workarounds for Termux-specific limitations and tips for getting the best performance.

---

## Table of Contents

- [Why Run ManusClaw on Termux?](#why-run-manusclaw-on-termux)
- [Prerequisites](#prerequisites)
- [Installing Termux](#installing-termux)
- [Initial Termux Setup](#initial-termux-setup)
- [Installing ManusClaw](#installing-manusclaw)
- [Using the Termux Setup Script](#using-the-termux-setup-script)
- [Configuring ManusClaw on Termux](#configuring-manusclaw-on-termux)
- [Running ManusClaw](#running-manusclaw)
- [Termux Limitations and Workarounds](#termux-limitations-and-workarounds)
- [Performance Optimization](#performance-optimization)
- [Always-On Setup](#always-on-setup)
- [Storage Access](#storage-access)
- [Troubleshooting Termux Issues](#troubleshooting-termux-issues)
- [Using an Android TV Box](#using-an-android-tv-box)

---

## Why Run ManusClaw on Termux?

You might wonder why anyone would run an AI agent framework on a phone. Here are some compelling reasons:

- **Portability** — Your AI agent is always with you, no laptop required. You can ask questions and run tasks on the go.
- **Low-cost always-on server** — An old Android phone can serve as a 24/7 ManusClaw server at zero additional cost. It uses far less electricity than a desktop or VPS.
- **Privacy** — All processing stays on your device. If you use Ollama with a small model, nothing leaves your phone.
- **Learning and experimentation** — Termux is a great way to learn about Linux, Python, and AI tools in a sandboxed environment.
- **Android TV boxes** — A $30 Android TV box with Termux can become a dedicated AI agent that's always running on your home network.

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Android version** | 7.0 (Nougat) or higher |
| **Storage** | At least 1 GB free (2 GB+ recommended) |
| **RAM** | 3 GB minimum (4 GB+ recommended) |
| **Architecture** | ARM64 (most modern phones), ARM, or x86_64 (emulators) |
| **Internet** | Required for API-based providers |

---

## Installing Termux

### ⚠️ Critical: Do NOT use the Google Play Store version

The Termux app on the Google Play Store is **outdated and broken**. Google requires apps to target a high API level, which breaks Termux's ability to execute programs. The Play Store version will not work for running Python or ManusClaw.

### Install from F-Droid (recommended)

1. Install [F-Droid](https://f-droid.org/) if you don't have it already
2. Search for "Termux" in F-Droid
3. Install Termux from F-Droid

### Install from GitHub (alternative)

1. Go to [github.com/termux/termux-app/releases](https://github.com/termux/termux-app/releases)
2. Download the appropriate APK for your device's architecture:
   - `termux-app_v0.118.0+github-debug_universal.apk` — Works on all devices
   - `termux-app_v0.118.0+github-debug_arm64-v8a.apk` — For ARM64 devices (most common)
   - `termux-app_v0.118.0+github-debug_armeabi-v7a.apk` — For older ARM devices
3. Install the APK (you may need to enable "Install from unknown sources" in Android settings)

### Termux:Float (optional)

If you want a floating Termux window that overlays other apps, also install Termux:Float from F-Droid. This is useful for quickly accessing ManusClaw while using other apps.

### Termux:API (optional)

Install Termux:API from F-Droid for access to Android hardware features (notifications, sensors, clipboard, etc.):

```bash
pkg install termux-api
```

---

## Initial Termux Setup

After installing Termux, follow these steps to prepare the environment for ManusClaw.

### Step 1: Update Termux packages

```bash
pkg update && pkg upgrade -y
```

When prompted about configuration file changes, press `Enter` to accept the default (keep the new version).

### Step 2: Install essential packages

```bash
pkg install -y python python-pip git build-essential binutils openssl portaudio
```

This installs:
- **python** — Python 3.11+ (Termux typically provides the latest stable version)
- **python-pip** — Package installer for Python
- **git** — Version control (needed for cloning repositories)
- **build-essential** — C compiler and build tools (needed for Python packages with C extensions)
- **binutils** — Binary utilities (needed by the build system)

### Step 3: Verify Python version

```bash
python --version
# Should show Python 3.11.x or higher

pip --version
# Should show pip 24.x or higher
```

### Step 4: Upgrade pip

```bash
pip install --upgrade pip
```

### Step 5: Grant storage access

To access your phone's shared storage (Downloads, Documents, etc.):

```bash
termux-setup-storage
```

This will prompt you for permission. Grant it, and a `~/storage/` directory will be created with symlinks to various shared storage locations:

```
~/storage/
├── shared/        → /sdcard/
├── downloads/     → /sdcard/Download/
├── dcim/          → /sdcard/DCIM/
├── music/         → /sdcard/Music/
├── pictures/      → /sdcard/Pictures/
└── movies/        → /sdcard/Movies/
```

### Step 6: Prevent Android from killing Termux

Android aggressively kills background processes to save battery. To prevent this:

```bash
# Acquire a wake lock (prevents Android from killing Termux in the background)
termux-wake-lock
```

You should see a notification that says "Termux is running." This is normal and expected. Without the wake lock, Android will kill Termux after a few minutes in the background.

For a more permanent solution:
1. Go to Android Settings → Apps → Termux → Battery
2. Select "Unrestricted" or "Don't optimize"
3. On some phones, also disable "Auto-start management" for Termux

---

## Installing ManusClaw

### Standard pip installation

```bash
pip install manusclaw
```

This may take a few minutes as it downloads and installs ManusClaw and all its dependencies. Some packages (like `pydantic-core` and `cryptography`) have C extensions that need to be compiled on-device, which can take 5-10 minutes on older phones.

### If you encounter build errors

Some dependencies may fail to build. Here are common fixes:

```bash
# Install additional build dependencies
pkg install -y libffi openssl rust portaudio
# Retry the installation
pip install manusclaw
```

If `cryptography` fails:

```bash
# Install cryptography separately first
pip install cryptography --no-binary :all:
pip install manusclaw
```

If `pydantic-core` fails:

```bash
# Install Rust compiler (required for pydantic-core)
pkg install rust
pip install pydantic-core
pip install manusclaw
```

### Installing from source

If pip installation doesn't work, try building from source:

```bash
git clone https://github.com/ManusClawAI/manusclaw.git
cd manusclaw
pip install -e .
```

---

## Using the Termux Setup Script

ManusClaw includes a dedicated `setup-termux.sh` script (previously `setup_termux.sh`) that automates the entire Termux installation process, including handling common issues:

```bash
# Clone the repository
git clone https://github.com/ManusClawAI/manusclaw.git
cd manusclaw

# Run the Termux setup script
bash setup-termux.sh
```

> **Note:** The script filename is `setup-termux.sh` (with a hyphen). If you encounter `setup_termux.sh` (underscore), it's the older name — use `setup-termux.sh` instead.

The script performs the following actions:
1. Updates Termux packages
2. Installs all required system packages (Python, build tools, etc.)
3. Installs ManusClaw and its Python dependencies
4. Creates the configuration directory and default files
5. Sets up the workspace directory
6. Configures wake lock for background operation
7. Runs a basic verification check

If the script encounters any issues, it provides clear error messages and suggested fixes.

---

## Configuring ManusClaw on Termux

### Setting up API keys

Since Termux doesn't have a graphical API key manager, you need to set keys via the command line:

```bash
# Create the config directory
mkdir -p ~/.manusclaw

# Create the .env file
cat > ~/.manusclaw/.env << 'EOF'
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
EOF

# Secure the file
chmod 600 ~/.manusclaw/.env
```

Or use environment variables directly:

```bash
echo 'export OPENAI_API_KEY="sk-proj-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"' >> ~/.bashrc
source ~/.bashrc
```

### Termux-optimized config.toml

Create a configuration optimized for the limited resources of a mobile device:

```bash
cat > ~/.manusclaw/config.toml << 'EOF'
# ManusClaw Configuration — Optimized for Termux

[llm]
provider = "openai"         # Use API-based providers (not Ollama)
model = "gpt-4o-mini"      # Smaller, faster, cheaper model
temperature = 0.7
max_tokens = 2048           # Reduced for mobile performance

[llm.openai]
api_key = ""                # Use OPENAI_API_KEY env var instead

[search]
engine = "duckduckgo"       # Playwright won't work on Termux
max_results = 5             # Fewer results to save bandwidth

[token_budget]
max_input_tokens = 32000    # Smaller context for mobile
max_output_tokens = 2048
max_total_tokens = 50000
auto_summarize = true       # Important for limited context

[permissions]
mode = "PLAN"               # Safer on mobile — confirm before acting

[workspace]
path = "workspace"
auto_create = true

[memory]
enabled = true
max_memory_size = 5000      # Smaller memory file for mobile

[logging]
level = "WARNING"           # Less verbose logging to save storage
file = ""
EOF
```

### Why these settings?

- **`gpt-4o-mini`** instead of `gpt-4o` — Smaller responses mean less data usage and faster results on mobile networks
- **`max_tokens = 2048`** — Reduces response time and token costs
- **`duckduckgo`** — Playwright (browser automation) doesn't work on Termux, but HTTP-based search works fine
- **`max_input_tokens = 32000`** — Smaller context uses less RAM, which is important on mobile devices
- **`WARNING` log level** — Reduces storage usage from log files
- **`PLAN` mode** — Safer on mobile where you might accidentally tap something

---

## Running ManusClaw

### Starting ManusClaw

```bash
# Make sure the wake lock is active
termux-wake-lock

# Start ManusClaw
manusclaw
```

### Using the interactive shell on a phone

Typing on a phone keyboard can be tedious. Here are some tips:

1. **Use a Bluetooth keyboard** — This is the most effective way to interact with ManusClaw on a phone
2. **Use voice-to-text** — Android's voice input works well with ManusClaw's natural language interface
3. **Use single-shot mode** — Quick questions are easier than long conversations:
   ```bash
   manusclaw "What is the current time in Tokyo?"
   ```
4. **Use SSH** — Connect to Termux from your computer:
   ```bash
   # In Termux:
   pkg install openssh
   passwd  # Set a password
   whoami  # Get your username
   ifconfig  # Get your IP address
   
   # Start the SSH server
   sshd
   
   # From your computer:
   ssh -p 8022 username@phone-ip-address
   ```

5. **Use Termux:Widget** — Create home screen shortcuts for common ManusClaw commands:
   ```bash
   mkdir -p ~/.shortcuts
   echo '#!/bin/bash
   manusclaw "Summarize today's calendar events"' > ~/.shortcuts/daily-brief
   chmod +x ~/.shortcuts/daily-brief
   ```

---

## Termux Limitations and Workarounds

### Playwright / Browser automation — NOT available

**Limitation:** Termux cannot run desktop browsers (Chromium, Firefox). This means the `web_browse` tool and any Playwright-dependent features will not work.

**Workaround:** Use DuckDuckGo search, which uses HTTP requests instead of a browser:

```toml
[search]
engine = "duckduckgo"
```

You can still search the web and fetch page content — you just can't interact with JavaScript-heavy pages or take screenshots.

### Voice features — Limited on Termux

**Limitation:** Termux/Android has limited audio device access. PyAudio may not be able to access the microphone or speakers, making v5 voice features (wake word, talk mode) unreliable.

**Workaround:** Use text-based interaction instead. If you need voice, use Termux's SSH to connect to a ManusClaw instance running on a desktop machine.

### SSH gateway — Use alternate port

**Limitation:** The v5 SSH gateway requires port 2222, which may conflict with Termux's own `sshd` running on port 8022.

**Workaround:** Use a different port for ManusClaw SSH:
```bash
export MANUSCLAW_SSH_PORT=2223
```

### Ollama — Limited support

**Limitation:** Running Ollama directly on Termux is possible but requires a device with significant RAM (6 GB+) and patience. The Android kernel doesn't support all the features that Ollama expects.

**Workaround:** Connect to an Ollama instance running on your computer or another server:

```toml
[llm.ollama]
base_url = "http://192.168.1.100:11434"  # Your computer's IP
model = "llama3"
```

If you want to try running Ollama on Termux:

```bash
# Install Ollama (experimental, may not work on all devices)
pkg install proot
git clone https://github.com/ollama/ollama
cd ollama
go build .
```

### Docker sandbox — NOT available

**Limitation:** Termux cannot run Docker or Docker Compose. This means the Docker sandbox backend for code isolation is unavailable.

**Workaround:** Use the `openshell` sandbox backend instead (namespace isolation) or set `SANDBOX_BACKEND=openshell` in your config.

### Background execution — Requires wake lock

**Limitation:** Android will kill Termux processes after a few minutes in the background.

**Workaround:**

```bash
# Acquire wake lock
termux-wake-lock

# Run ManusClaw server
manusclaw-server --host 0.0.0.0 --port 8765 &

# Keep Termux in the foreground as a notification
# The wake lock notification tells Android not to kill the process
```

### Limited disk space

**Limitation:** Termux uses a portion of your phone's internal storage, which may be limited.

**Workaround:**

```bash
# Check available space
df -h /data

# Clean up pip cache
pip cache purge

# Clean up Termux package cache
pkg clean

# Use SD card storage (if available)
# Move workspace to shared storage
ln -s ~/storage/shared/manusclaw-workspace ~/workspace
```

---

## Performance Optimization

### Reduce memory usage

```bash
# Use a Python virtual environment to avoid installing globally
python -m venv ~/mc-env
source ~/mc-env/bin/activate
pip install manusclaw
```

### Reduce network usage

```toml
[llm]
max_tokens = 1024           # Shorter responses
model = "gpt-4o-mini"       # Cheaper, faster model

[search]
max_results = 3             # Fewer search results

[token_budget]
max_input_tokens = 16000    # Smaller context
auto_summarize = true
```

### Use Termux on a Chromebook

If you have a Chromebook that supports Android apps, you can run Termux with a full keyboard and larger screen:

1. Install Termux from the Google Play Store (Chromebooks are exempt from the API level issue)
2. Or use the Linux (Beta) feature on Chrome OS for a native Linux environment

---

## Always-On Setup

To run ManusClaw as an always-available service on your phone:

### Step 1: Create a startup script

```bash
cat > ~/start_manusclaw.sh << 'EOF'
#!/bin/bash
# Start ManusClaw server on Android

# Prevent Android from killing the process
termux-wake-lock

# Activate virtual environment (if using one)
source ~/mc-env/bin/activate 2>/dev/null

# Start the server
manusclaw-server --host 0.0.0.0 --port 8765

# If the server crashes, restart after a delay
while true; do
    echo "ManusClaw server crashed. Restarting in 10 seconds..."
    sleep 10
    manusclaw-server --host 0.0.0.0 --port 8765
done
EOF

chmod +x ~/start_manusclaw.sh
```

### Step 2: Auto-start on Termux launch

```bash
# Add to ~/.bashrc
echo '~/start_manusclaw.sh &' >> ~/.bashrc
```

### Step 3: Auto-start on boot (requires Termux:Boot)

1. Install [Termux:Boot](https://f-droid.org/packages/com.termux.boot/) from F-Droid
2. Create the boot script directory:
   ```bash
   mkdir -p ~/.termux/boot
   ```
3. Create a boot script:
   ```bash
   cat > ~/.termux/boot/manusclaw.sh << 'EOF'
   #!/bin/bash
   termux-wake-lock
   source ~/mc-env/bin/activate 2>/dev/null
   manusclaw-server --host 0.0.0.0 --port 8765 &
   EOF
   chmod +x ~/.termux/boot/manusclaw.sh
   ```
4. Open Termux:Boot app once to enable it
5. ManusClaw will now start automatically when your phone boots

### Step 4: Access from other devices

Once the server is running, access it from any device on the same network:

```bash
# Find your phone's IP address
ifconfig wlan0 | grep inet

# From another device:
curl http://192.168.1.x:8765/health
```

---

## Storage Access

### Reading files from your phone

```bash
# Access your Downloads folder
ls ~/storage/downloads/

# Read a file from shared storage
manusclaw "Summarize this document" < ~/storage/downloads/report.pdf

# Set workspace to shared storage
ln -s ~/storage/shared/my-project ~/workspace
```

### Writing files to shared storage

```bash
# ManusClaw can write to the workspace directory
# If workspace is a symlink to shared storage, files appear in your phone's file manager

# Or write directly
cp workspace/output.txt ~/storage/downloads/
```

---

## Troubleshooting Termux Issues

### Python crashes with segfault

This can happen on devices with limited RAM or incompatible kernels:

```bash
# Reduce Python's memory usage
export PYTHONHASHSEED=0
export PYTHONDONTWRITEBYTECODE=1

# Use a smaller model and fewer tokens
manusclaw --provider openai --model gpt-4o-mini
```

### `pkg` command not found

You might be using an old or incorrect version of Termux:

```bash
# Update Termux packages
apt update && apt upgrade -y

# If apt is not found, you're likely using the Play Store version
# Uninstall and install from F-Droid or GitHub
```

### Cannot install packages (connection refused)

Termux repositories may be blocked in some regions:

```bash
# Switch to a mirror
termux-change-repo

# Or manually set a mirror
echo "deb https://mirrors.cqu.edu.cn/termux/termux-packages stable main" > /data/data/com.termux/files/usr/etc/apt/sources.list
apt update
```

### Keyboard issues

The Termux keyboard can be limited on some devices:

1. **Enable the extra keys row:**
   - Long-press the keyboard icon in the Termux notification
   - Or swipe from the left edge to open the drawer → Keyboard → Extra keys

2. **Common key combinations:**
   - `Ctrl+C`: Volume Down + C
   - `Ctrl+D`: Volume Down + D
   - `Tab`: Volume Down + T (or swipe right on the keyboard)
   - `Esc`: Volume Down + E (or swipe left on the keyboard)
   - `Up/Down arrows`: Volume Down + W/S

3. **Install Hacker's Keyboard** from F-Droid for a full PC-style keyboard with Ctrl, Tab, and arrow keys.

---

## Using an Android TV Box

Android TV boxes (like Xiaomi Mi Box, Fire TV Stick, or generic boxes) make excellent always-on ManusClaw servers because they:
- Are cheap ($20-50)
- Have Ethernet ports (more reliable than Wi-Fi)
- Are always plugged in (no battery concerns)
- Can be accessed via SSH from other devices

### Setup instructions

1. **Install Termux** — Sideload the F-Droid APK, then install Termux from F-Droid
2. **Connect via ADB** (easier than using the remote control):
   ```bash
   # From your computer:
   adb connect android-tv-ip:5555
   adb shell
   ```
3. **Follow the standard Termux installation** instructions from this guide
4. **Set up SSH access:**
   ```bash
   pkg install openssh
   passwd
   sshd
   ```
5. **Configure as an always-on server** following the Always-On Setup section above
6. **Access from your network:**
   ```bash
   ssh -p 8022 username@android-tv-ip
   manusclaw
   ```
