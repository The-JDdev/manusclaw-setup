# Termux (Android) Guide — ManusClaw v5.1.0

Running ManusClaw on Android via Termux transforms your phone or tablet into a portable AI agent. This guide covers everything you need to know about installing, configuring, and using ManusClaw v5.1.0 on Android, including workarounds for Termux-specific limitations and tips for getting the best performance.

---

## Table of Contents

- [Why Run ManusClaw on Termux?](#why-run-manusclaw-on-termux)
- [Prerequisites](#prerequisites)
- [Installing Termux](#installing-termux)
- [Initial Termux Setup](#initial-termux-setup)
- [Installing ManusClaw](#installing-manusclaw)
- [Using the Termux Setup Script](#using-the-termux-setup-script)
- [Configuring ManusClaw on Termux](#configuring-manusclaw-on-termux)
- [v5.1 Features on Termux](#v51-features-on-termux)
- [Running ManusClaw](#running-manusclaw)
- [Termux Limitations and Workarounds](#termux-limitations-and-workarounds)
- [Performance Optimization](#performance-optimization)
- [Always-On Setup](#always-on-setup)
- [Storage Access](#storage-access)
- [Troubleshooting Termux Issues](#troubleshooting-termux-issues)
- [Using an Android TV Box](#using-an-android-tv-box)

---

## Why Run ManusClaw on Termux?

- **Portability** — Your AI agent is always with you, no laptop required
- **Low-cost always-on server** — An old Android phone can serve as a 24/7 ManusClaw server at zero additional cost
- **Privacy** — All processing stays on your device when using Ollama with a small model
- **Learning and experimentation** — Termux is a great way to learn about Linux, Python, and AI tools
- **Android TV boxes** — A $30 Android TV box with Termux can become a dedicated AI agent

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

The Termux app on the Google Play Store is **outdated and broken**. Google requires apps to target a high API level, which breaks Termux's ability to execute programs.

### Install from F-Droid (recommended)

1. Install [F-Droid](https://f-droid.org/) if you don't have it already
2. Search for "Termux" in F-Droid
3. Install Termux from F-Droid

### Install from GitHub (alternative)

1. Go to [github.com/termux/termux-app/releases](https://github.com/termux/termux-app/releases)
2. Download the appropriate APK for your device's architecture
3. Install the APK (you may need to enable "Install from unknown sources")

### Termux:Float (optional)

Floating Termux window that overlays other apps — useful for quick ManusClaw access.

### Termux:API (optional)

Access Android hardware features (notifications, sensors, clipboard):

```bash
pkg install termux-api
```

---

## Initial Termux Setup

### Step 1: Update Termux packages

```bash
pkg update && pkg upgrade -y
```

### Step 2: Install essential packages

```bash
pkg install -y python python-pip git build-essential binutils openssl portaudio
```

### Step 3: Verify Python version

```bash
python --version
# Should show Python 3.11.x or higher

pip --version
```

### Step 4: Upgrade pip

```bash
pip install --upgrade pip
```

### Step 5: Grant storage access

```bash
termux-setup-storage
```

This creates `~/storage/` with symlinks to shared storage locations.

### Step 6: Prevent Android from killing Termux

```bash
termux-wake-lock
```

Also: Android Settings → Apps → Termux → Battery → "Unrestricted"

---

## Installing ManusClaw

### Standard pip installation

```bash
pip install manusclaw
```

Some packages with C extensions may take 5-10 minutes to compile on older phones.

### If you encounter build errors

```bash
# Install additional build dependencies
pkg install -y libffi openssl rust portaudio
pip install manusclaw
```

If `cryptography` fails:

```bash
pip install cryptography --no-binary :all:
pip install manusclaw
```

If `pydantic-core` fails:

```bash
pkg install rust
pip install pydantic-core
pip install manusclaw
```

### Installing with v5.1 optional groups

```bash
# Core + voice
pip install "manusclaw[voice]"

# Core + channels
pip install "manusclaw[channels]"

# Everything (may be slow on older devices)
pip install "manusclaw[all]"

# Skip enterprise features (Vault, observability) on mobile
pip install "manusclaw[voice,channels,browser]"
```

### Installing from source

```bash
git clone https://github.com/ManusAgents/manusclaw.git
cd manusclaw
pip install -e .
```

---

## Using the Termux Setup Script

ManusClaw includes a dedicated `setup-termux.sh` script:

```bash
git clone https://github.com/ManusAgents/manusclaw.git
cd manusclaw
bash setup-termux.sh
```

The script performs the following actions:
1. Updates Termux packages
2. Installs all required system packages
3. Installs ManusClaw and its Python dependencies
4. Creates the configuration directory and default files
5. Sets up the workspace directory
6. Configures wake lock for background operation
7. Runs a basic verification check

---

## Configuring ManusClaw on Termux

### Setting up API keys

```bash
mkdir -p ~/.manusclaw

cat > ~/.manusclaw/.env << 'EOF'
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
EOF

chmod 600 ~/.manusclaw/.env
```

### Termux-optimized config

```bash
cat > ~/.manusclaw/config.toml << 'EOF'
# ManusClaw Configuration — Optimized for Termux

[llm]
provider = "openai"
model = "gpt-4o-mini"
temperature = 0.7
max_tokens = 2048

[llm.openai]
api_key = ""

[search]
engine = "duckduckgo"
max_results = 5

[token_budget]
max_input_tokens = 32000
max_output_tokens = 2048
max_total_tokens = 50000
auto_summarize = true

[permissions]
mode = "PLAN"

[workspace]
path = "workspace"
auto_create = true

[memory]
enabled = true
max_memory_size = 5000

[logging]
level = "WARNING"
file = ""
EOF
```

### v5.1 config.yaml for Termux

```bash
cat > ~/.manusclaw/config.yaml << 'EOF'
# ManusClaw v5.1 — Termux-optimized config

model_profiles:
  default:
    - provider: openai
      model: gpt-4o-mini
      priority: 1
    - provider: groq
      model: llama-3.3-70b-versatile
      priority: 2

security:
  rbac:
    enabled: false
  input_validation:
    enabled: true
    max_input_length: 5000
  rate_limiting:
    enabled: true
    requests_per_minute: 30

context:
  compression:
    enabled: true
    strategy: "sliding"
    sliding:
      keep_recent_messages: 10

conversation:
  persistence:
    enabled: true
    backend: "file"

observability:
  enabled: false  # Disable to save resources

secrets:
  backend: "env"  # Simple env vars on mobile

file_store:
  backend: "local"
  local:
    base_path: "~/.manusclaw/files"
    max_file_size_mb: 50

parallel_executor:
  enabled: false  # Limited CPU on mobile

EOF
```

---

## v5.1 Features on Termux

### Feature Availability

| Feature | Available | Notes |
|---------|-----------|-------|
| Single-shot mode | ✅ | Primary usage |
| Server mode | ✅ | With wake lock |
| Channels | ✅ | All channel adapters work |
| Webhooks | ✅ | Via server |
| Canvas (WebChat) | ✅ | Via WebSocket |
| Cron scheduling | ✅ | With wake lock |
| Voice (wake/talk) | ⚠️ | PyAudio may not work reliably |
| SSH Gateway | ⚠️ | Use alternate port (2223) |
| Security / RBAC | ✅ | Server-side |
| Hooks | ✅ | Shell hooks recommended |
| Context management | ✅ | Use sliding strategy |
| Observability | ⚠️ | Disable to save resources |
| Secrets (Vault) | ❌ | No Vault on mobile; use env |
| File store (S3) | ✅ | Good for persistence |
| Git providers | ✅ | API-based |
| Parallel executor | ⚠️ | Limited CPU; use threaded with max_workers=2 |
| Migrations | ✅ | Config migrations |
| Integrations | ⚠️ | Limited by network/resources |

### Using Hooks on Termux

Shell hooks work well on Termux since you have a full shell environment:

```bash
# Create a hook script
mkdir -p ~/.manusclaw/hooks

cat > ~/.manusclaw/hooks/pre_execute.sh << 'SCRIPT'
#!/bin/bash
# Log agent activity to a file
echo "[$(date -Iseconds)] Agent ${MANUSCLAW_AGENT} executing: ${MANUSCLAW_PROMPT:0:80}" \
  >> ~/.manusclaw/logs/hook.log
exit 0
SCRIPT

chmod +x ~/.manusclaw/hooks/pre_execute.sh
```

Add to `config.yaml`:

```yaml
hooks:
  pre_execute:
    - name: "log_request"
      type: "shell"
      command: "~/.manusclaw/hooks/pre_execute.sh"
      timeout: 10
      on_failure: "warn"
```

### Using File Store with S3

For persistent file storage that survives runtime disconnects:

```bash
pip install "manusclaw[cloud]"

# Configure S3
export AWS_ACCESS_KEY_ID=your-key
export AWS_SECRET_ACCESS_KEY=your-secret
export AWS_DEFAULT_REGION=us-east-1
```

---

## Running ManusClaw

### Starting ManusClaw

```bash
termux-wake-lock
manusclaw
```

### Using the interactive shell on a phone

1. **Use a Bluetooth keyboard** — Most effective for Termux interaction
2. **Use voice-to-text** — Android's voice input works with ManusClaw's natural language interface
3. **Use single-shot mode** — Quick questions are easier than long conversations:
   ```bash
   manusclaw "What is the current time in Tokyo?"
   ```
4. **Use SSH** — Connect to Termux from your computer:
   ```bash
   pkg install openssh
   passwd
   sshd
   # From your computer:
   ssh -p 8022 username@phone-ip-address
   ```
5. **Use Termux:Widget** — Create home screen shortcuts:
   ```bash
   mkdir -p ~/.shortcuts
   echo '#!/bin/bash
   manusclaw "Summarize today's calendar events"' > ~/.shortcuts/daily-brief
   chmod +x ~/.shortcuts/daily-brief
   ```

---

## Termux Limitations and Workarounds

### Playwright / Browser automation — NOT available

**Workaround:** Use DuckDuckGo search:

```toml
[search]
engine = "duckduckgo"
```

### Voice features — Limited on Termux

**Workaround:** Use text-based interaction. If you need voice, connect to a ManusClaw desktop instance via SSH.

### SSH gateway — Use alternate port

**Workaround:**

```bash
export MANUSCLAW_SSH_PORT=2223
```

### Ollama — Limited support

**Workaround:** Connect to a remote Ollama instance:

```toml
[llm.ollama]
base_url = "http://192.168.1.100:11434"
model = "llama3"
```

### Docker sandbox — NOT available

**Workaround:** Use the `openshell` sandbox backend:

```bash
export SANDBOX_BACKEND=openshell
```

### Background execution — Requires wake lock

```bash
termux-wake-lock
manusclaw-server --host 0.0.0.0 --port 8765 &
```

### v5.1: Vault — NOT available on Termux

**Workaround:** Use `env` or `encrypted_file` secrets backend:

```yaml
secrets:
  backend: "env"
  # or
  encrypted_file:
    enabled: true
    path: "~/.manusclaw/secrets/secrets.enc"
```

### v5.1: Observability — Resource intensive

**Workaround:** Disable observability on mobile:

```yaml
observability:
  enabled: false
```

Use structured logging to a file instead if needed:

```yaml
observability:
  logging:
    structured: true
    format: "json"
    output:
      - type: "file"
        path: "~/.manusclaw/logs/manusclaw.jsonl"
```

### Limited disk space

```bash
df -h /data
pip cache purge
pkg clean
ln -s ~/storage/shared/manusclaw-workspace ~/workspace
```

---

## Performance Optimization

### Reduce memory usage

```bash
python -m venv ~/mc-env
source ~/mc-env/bin/activate
pip install manusclaw
```

### Reduce network usage

```toml
[llm]
max_tokens = 1024
model = "gpt-4o-mini"

[search]
max_results = 3

[token_budget]
max_input_tokens = 16000
auto_summarize = true
```

### Use Termux on a Chromebook

If you have a Chromebook that supports Android apps, you can run Termux with a full keyboard and larger screen.

---

## Always-On Setup

### Step 1: Create a startup script

```bash
cat > ~/start_manusclaw.sh << 'EOF'
#!/bin/bash
termux-wake-lock
source ~/mc-env/bin/activate 2>/dev/null
manusclaw-server --host 0.0.0.0 --port 8765
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
   cat > ~/.termux/boot/manusclaw.sh << 'SCRIPT'
   #!/bin/bash
   termux-wake-lock
   source ~/mc-env/bin/activate 2>/dev/null
   manusclaw-server --host 0.0.0.0 --port 8765 &
   SCRIPT
   chmod +x ~/.termux/boot/manusclaw.sh
   ```
4. Open Termux:Boot app once to enable it
5. ManusClaw will now start automatically when your phone boots

### Step 4: Access from other devices

```bash
ifconfig wlan0 | grep inet
# From another device:
curl http://192.168.1.x:8765/health
```

---

## Storage Access

### Reading files from your phone

```bash
ls ~/storage/downloads/
manusclaw "Summarize this document" < ~/storage/downloads/report.pdf
ln -s ~/storage/shared/my-project ~/workspace
```

### Writing files to shared storage

```bash
cp workspace/output.txt ~/storage/downloads/
```

---

## Troubleshooting Termux Issues

### Python crashes with segfault

```bash
export PYTHONHASHSEED=0
export PYTHONDONTWRITEBYTECODE=1
manusclaw --provider openai --model gpt-4o-mini
```

### `pkg` command not found

```bash
apt update && apt upgrade -y
# If apt is not found, you're likely using the Play Store version
# Uninstall and install from F-Droid or GitHub
```

### Cannot install packages (connection refused)

```bash
termux-change-repo
# Or manually set a mirror
```

### Keyboard issues

1. **Enable the extra keys row:** Long-press the keyboard icon in the Termux notification
2. **Common key combinations:**
   - `Ctrl+C`: Volume Down + C
   - `Ctrl+D`: Volume Down + D
   - `Tab`: Volume Down + T
   - `Esc`: Volume Down + E

### v5.1: Parallel executor uses too much CPU

```bash
# Limit workers to 2 on mobile
manusclaw config set parallel_executor.workers.max_workers 2
manusclaw config set parallel_executor.mode threaded
```

### v5.1: Context window causing memory issues

```bash
# Use sliding compression with fewer kept messages
manusclaw config set context.compression.strategy sliding
manusclaw config set context.compression.sliding.keep_recent_messages 10
```

---

## Using an Android TV Box

Android TV boxes make excellent always-on ManusClaw servers because they:
- Are cheap ($20-50)
- Have Ethernet ports (more reliable than Wi-Fi)
- Are always plugged in (no battery concerns)
- Can be accessed via SSH from other devices

### Setup instructions

1. **Install Termux** — Sideload the F-Droid APK, then install Termux from F-Droid
2. **Connect via ADB** (easier than using the remote control):
   ```bash
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
