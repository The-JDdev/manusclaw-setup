# Troubleshooting Guide — ManusClaw v5.0.0

This guide covers the most common issues you might encounter when installing, configuring, or running ManusClaw. Each problem includes a detailed explanation of the root cause and step-by-step solutions. If your issue isn't covered here, please open a GitHub issue with the details of your environment and the error message you're seeing.

---

## Table of Contents

- [Common Installation Errors](#common-installation-errors)
- [Python Version Issues](#python-version-issues)
- [pip / Dependency Conflicts](#pip--dependency-conflicts)
- [API Key Configuration Problems](#api-key-configuration-problems)
- [Ollama Connection Issues](#ollama-connection-issues)
- [Playwright Browser Issues](#playwright-browser-issues)
- [Voice & Audio Issues (v5)](#voice--audio-issues-v5)
- [SSH Gateway Issues (v5)](#ssh-gateway-issues-v5)
- [Channel Adapter Issues (v5)](#channel-adapter-issues-v5)
- [Webhook Issues (v5)](#webhook-issues-v5)
- [Model Failover Issues (v5)](#model-failover-issues-v5)
- [Credential Pool Issues (v5)](#credential-pool-issues-v5)
- [Gmail Pub/Sub Issues (v5)](#gmail-pubsub-issues-v5)
- [Permission Denied Errors](#permission-denied-errors)
- [Memory / Database Errors](#memory--database-errors)
- [Network / Firewall Issues](#network--firewall-issues)
- [Windows-Specific Issues](#windows-specific-issues)
- [Termux-Specific Issues](#termux-specific-issues)
- [Docker-Specific Issues](#docker-specific-issues)
- [Rate Limiting Issues](#rate-limiting-issues)
- [Token Budget Exhausted](#token-budget-exhausted)
- [How to Reset / Clean Install](#how-to-reset--clean-install)

---

## Common Installation Errors

### Error: `command not found: manusclaw`

**Symptom:** After installing ManusClaw with `pip install manusclaw`, the `manusclaw` command is not recognized in your terminal.

**Root Cause:** The pip binary directory is not in your system's PATH. This typically happens when:
- You installed Python via pyenv, Homebrew, or a custom location
- You used `pip install --user` and the user bin directory isn't in PATH
- You're using a virtual environment that hasn't been activated

**Solution:**

1. Find where pip installed the manusclaw binary:
   ```bash
   python3 -m pip show manusclaw | grep Location
   # Or:
   which manusclaw
   find ~/.local -name manusclaw 2>/dev/null
   ```

2. Add the correct directory to your PATH. The most common locations are:
   ```bash
   # For --user installs
   echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
   source ~/.bashrc

   # For Homebrew on macOS (Apple Silicon)
   echo 'export PATH="/opt/homebrew/bin:$PATH"' >> ~/.zshrc
   source ~/.zshrc

   # For pyenv
   echo 'export PATH="$HOME/.pyenv/shims:$HOME/.pyenv/bin:$PATH"' >> ~/.bashrc
   source ~/.bashrc
   ```

3. If you're using a virtual environment, make sure it's activated:
   ```bash
   source ~/manusclaw-env/bin/activate
   manusclaw --version
   ```

4. As a last resort, run ManusClaw directly through Python:
   ```bash
   python3 -m manusclaw
   ```

---

### Error: `error: externally-managed-environment`

**Symptom:** On Ubuntu 23.04+ or Fedora 38+, `pip install` fails with:
```
error: externally-managed-environment
× This environment is externally managed
╰─> To install Python packages system-wide, try apt install
    python3-xyz, where xyz is the package you want to install.
```

**Root Cause:** Recent versions of Ubuntu and Fedora use PEP 668, which prevents pip from installing packages into the system Python to avoid conflicts with the OS's package manager. This is a good thing — it protects your system Python — but it can be confusing.

**Solution (recommended): Use a virtual environment:**

```bash
# Create a virtual environment
python3 -m venv ~/manusclaw-env

# Activate it
source ~/manusclaw-env/bin/activate

# Install ManusClaw
pip install manusclaw

# Run ManusClaw
manusclaw --version
```

To automatically activate the virtual environment when you open a terminal:

```bash
echo 'source ~/manusclaw-env/bin/activate' >> ~/.bashrc
```

**Alternative (not recommended): Override with --break-system-packages:**

```bash
pip install --break-system-packages manusclaw
```

This works but may cause conflicts with system Python packages. Use only if you understand the risks.

---

### Error: `Failed building wheel for ...`

**Symptom:** During `pip install manusclaw`, you see errors about failing to build a C extension wheel, often for packages like `cffi`, `cryptography`, or `pydantic-core`.

**Root Cause:** Some of ManusClaw's dependencies have C extensions that need to be compiled during installation. This requires a C compiler and development headers.

**Solution:**

Install the necessary build tools:

**Ubuntu/Debian:**
```bash
sudo apt install -y build-essential python3-dev libssl-dev libffi-dev portaudio19-dev
```

**Fedora/RHEL:**
```bash
sudo dnf install -y gcc gcc-c++ python3-devel openssl-devel libffi-devel portaudio-devel
```

**macOS:**
```bash
xcode-select --install
brew install portaudio
```

**Arch:**
```bash
sudo pacman -S --needed base-devel python python-pip portaudio
```

After installing the build tools, retry:

```bash
pip install manusclaw
```

---

## Python Version Issues

### Error: `ManusClaw requires Python 3.11 or newer`

**Symptom:** You get an error during installation or runtime indicating that your Python version is too old.

**Diagnosis:**

```bash
python3 --version
python --version
```

If either shows Python 3.10 or older, you need to upgrade. ManusClaw v5.0.0 requires **Python 3.11+**.

**Solution:**

See the [Installation Guide](installation.md) for platform-specific instructions on installing Python 3.11+. Here's a quick reference:

**Ubuntu (using deadsnakes PPA):**
```bash
sudo add-apt-repository -y ppa:deadsnakes/ppa
sudo apt update
sudo apt install -y python3.11 python3.11-venv python3.11-dev
python3.11 -m venv ~/manusclaw-env
source ~/manusclaw-env/bin/activate
pip install manusclaw
```

**Using pyenv (any Linux/macOS):**
```bash
curl https://pyenv.run | bash
# Restart your shell, then:
pyenv install 3.12.4
pyenv global 3.12.4
pip install manusclaw
```

**macOS (using Homebrew):**
```bash
brew install python@3.12
pip3.12 install manusclaw
```

### Error: Multiple Python versions causing confusion

**Symptom:** `python3 --version` shows 3.11+ but pip installs to a different Python version.

**Diagnosis:**
```bash
# Check which Python pip is using
pip --version
# Should show: pip 24.x from /path/to/python3.11/...

# If it shows a different version, pip is linked to the wrong Python
```

**Solution:**

Always use the explicit Python version when installing:
```bash
python3.11 -m pip install manusclaw
# Or:
python3.12 -m pip install manusclaw
```

Or use a virtual environment, which isolates the Python version:
```bash
python3.11 -m venv myenv
source myenv/bin/activate
pip install manusclaw
```

---

## pip / Dependency Conflicts

### Error: `ERROR: Cannot install manusclaw because these package versions have conflicting dependencies`

**Symptom:** pip reports a dependency conflict, often involving `pydantic`, `openai`, or `httpx`.

**Root Cause:** You may have other Python packages installed that require incompatible versions of shared dependencies. This is common in shared environments or when you've installed many packages globally.

**Solution 1: Use a fresh virtual environment**

This is almost always the best solution. A fresh virtual environment has no pre-existing packages, so there can't be conflicts:

```bash
python3 -m venv ~/fresh-manusclaw
source ~/fresh-manusclaw/bin/activate
pip install --upgrade pip
pip install manusclaw
```

**Solution 2: Update pip and setuptools**

Older versions of pip are worse at resolving dependencies:

```bash
pip install --upgrade pip setuptools wheel
pip install manusclaw
```

**Solution 3: Install with --force-reinstall**

If you must install globally:

```bash
pip install --force-reinstall manusclaw
```

**Solution 4: Check for conflicting packages**

```bash
pip check
```

This will list any installed packages with broken dependencies.

---

## API Key Configuration Problems

### Error: `openai.AuthenticationError: Incorrect API key provided`

**Symptom:** ManusClaw fails to communicate with the LLM provider, reporting an authentication error.

**Root Cause:** The API key is missing, incorrect, or not being loaded from the expected location.

**Diagnosis:**

```bash
# Check if the environment variable is set
echo $OPENAI_API_KEY

# Check if the .env file exists and contains the key
cat ~/.manusclaw/.env | grep OPENAI_API_KEY

# Check the config.toml
grep -A5 "\[llm.openai\]" ~/.manusclaw/config.toml
```

**Solutions:**

1. **Set the environment variable directly:**
   ```bash
   export OPENAI_API_KEY="sk-proj-your-actual-key"
   manusclaw
   ```

2. **Add it to the .env file:**
   ```bash
   echo 'OPENAI_API_KEY=sk-proj-your-actual-key' >> ~/.manusclaw/.env
   ```

3. **Add it to config.toml:**
   ```toml
   [llm.openai]
   api_key = "sk-proj-your-actual-key"
   ```

4. **Check for common mistakes:**
   - Extra whitespace around the key
   - Missing the `sk-proj-` prefix (for OpenAI keys)
   - Using an old key format (OpenAI changed their key format in 2023)
   - The key has been revoked or expired

### Error: `openai.RateLimitError: You exceeded your current quota`

**Symptom:** You can authenticate but requests are rejected due to billing/quota issues.

**Solutions:**
1. Check your billing status at [platform.openai.com/account/billing](https://platform.openai.com/account/billing)
2. Add billing information if you haven't already
3. Wait if you've hit a usage limit (limits reset periodically)
4. Use a different provider or add more API keys to the credential pool (see [Credential Pool Issues](#credential-pool-issues-v5))

### Error: Provider-specific key format issues

Each provider has a different API key format:

| Provider | Key Format | Example |
|----------|-----------|---------|
| OpenAI | `sk-proj-...` or `sk-...` | `sk-proj-abc123...` |
| Anthropic | `sk-ant-...` | `sk-ant-api03-abc123...` |
| Google | `AIzaSy...` | `AIzaSyB-abc123...` |
| Mistral | No prefix, 32+ chars | `abc123def456...` |
| HuggingFace | `hf_...` | `hf_abc123def456...` |
| OpenRouter | `sk-or-...` | `sk-or-v1-abc123...` |
| Groq | `gsk_...` | `gsk_abc123...` |

If your key doesn't match the expected format, double-check that you copied it correctly from the provider's dashboard.

---

## Ollama Connection Issues

### Error: `ConnectionRefusedError: Cannot connect to Ollama at localhost:11434`

**Symptom:** ManusClaw cannot connect to the Ollama server.

**Diagnosis:**

```bash
# Check if Ollama is running
curl http://localhost:11434/api/tags

# Check if Ollama process is active
ps aux | grep ollama

# Check the port
ss -tlnp | grep 11434
```

**Solutions:**

1. **Start the Ollama server:**
   ```bash
   ollama serve
   ```

2. **Check if Ollama is on a different port:**
   ```bash
   # Check Ollama's configuration
   echo $OLLAMA_HOST
   ```

3. **If Ollama is on a remote server, update your config:**
   ```toml
   [llm.ollama]
   base_url = "http://your-server-ip:11434"
   ```

4. **If using Ollama in Docker, ensure the containers are on the same network:**
   ```bash
   docker network inspect manusclaw-net
   ```

### Error: Model not found in Ollama

**Symptom:** `Error: model "llama3" not found, try pulling it first`

**Solution:**

```bash
# List available models
ollama list

# Pull the model you need
ollama pull llama3

# Verify it's available
ollama list | grep llama3
```

### Error: Ollama is running but very slow

**Symptom:** Responses take a very long time (>60 seconds) when using Ollama.

**Diagnosis:**

```bash
# Check resource usage
htop

# Check if Ollama is using GPU
nvidia-smi
```

**Solutions:**

1. **Use a smaller model** (e.g., `phi3` instead of `llama3:70b`)
2. **Enable GPU acceleration** (requires NVIDIA GPU with CUDA)
3. **Increase timeout** in your config:
   ```toml
   [llm.ollama]
   timeout = 600  # 10 minutes
   ```
4. **Close other resource-intensive applications**
5. **Use a quantized model** (smaller, faster, slightly less accurate)

---

## Playwright Browser Issues

### Error: `playwright._impl._errors.Error: BrowserType.launch: Executable doesn't exist`

**Symptom:** ManusClaw can't launch a browser for web browsing tasks.

**Root Cause:** Playwright browsers haven't been installed, or the installation is corrupted.

**Solution:**

```bash
# Install Chromium (the only browser ManusClaw needs)
playwright install chromium

# Install with system dependencies (resolves most issues on Linux)
playwright install --with-deps chromium

# If that doesn't work, try a clean reinstall
pip uninstall playwright -y
pip install playwright
playwright install --with-deps chromium
```

### Error: Missing shared libraries (Linux)

**Symptom:** Playwright fails with errors like:
```
error while loading shared libraries: libgbm.so.1: cannot open shared object file
```

**Root Cause:** Playwright's Chromium requires several system libraries that may not be installed on minimal Linux distributions (Docker containers, WSL, headless servers).

**Solution:**

```bash
# Use --with-deps to install system dependencies automatically
playwright install --with-deps chromium

# Or manually install the dependencies:

# Ubuntu/Debian
sudo apt install -y libnss3 libnspr4 libatk1.0-0 libatk-bridge2.0-0 \
    libcups2 libdrm2 libxkbcommon0 libxcomposite1 libxdamage1 \
    libxrandr2 libgbm1 libpango-1.0-0 libcairo2 libasound2 \
    libxshmfence1

# Fedora/RHEL
sudo dnf install -y nss nspr atk at-spi2-atk cups-libs libdrm \
    libXcomposite libXdamage libXrandr mesa-libgbm pango cairo \
    alsa-lib
```

### Error: Playwright doesn't work in Docker

**Solution:** Use a Docker image that includes the necessary dependencies. Add this to your Dockerfile:

```dockerfile
# Install Playwright system dependencies
RUN playwright install --with-deps chromium
```

Or use the official Playwright Docker image as a base:

```dockerfile
FROM mcr.microsoft.com/playwright/python:v1.40.0-jammy
```

### Error: Playwright doesn't work on Termux

**Root Cause:** Termux does not support running Chromium or other desktop browsers. This is a fundamental limitation of the Android/Termux environment.

**Workaround:** Web browsing via Playwright is not available on Termux. However, DuckDuckGo search (which uses HTTP requests, not a browser) works fine. Configure your search to not use Playwright:

```toml
[search]
engine = "duckduckgo"
```

---

## Voice & Audio Issues (v5)

### Error: Voice features not working — `No module named 'pyaudio'`

**Symptom:** `manusclaw voice wake` or `manusclaw voice talk` fails with a PyAudio import error.

**Root Cause:** PyAudio requires PortAudio system libraries, which are not installed by default.

**Solution:**

**Ubuntu/Debian:**
```bash
sudo apt install -y portaudio19-dev
pip install pyaudio
```

**macOS:**
```bash
brew install portaudio
pip install pyaudio
```

**Fedora/RHEL:**
```bash
sudo dnf install -y portaudio-devel
pip install pyaudio
```

**WSL2:** See the [WSL2 Guide](platforms/wsl.md) for audio device passthrough notes.

### Error: Porcupine wake word not detected

**Symptom:** `manusclaw voice wake` starts but never detects the wake word.

**Diagnosis:**

1. **Check if Porcupine key is set:**
   ```bash
   echo $PICOVOICE_API_KEY
   ```

2. **Check the log output for which backend is active:**
   ```bash
   manusclaw voice wake --start --word "hey manus" --debug
   ```

**Solutions:**

- **Without a Porcupine key:** The system falls back to Google STT, which has ~2s latency and requires internet. It uses substring matching on audio transcripts.
- **With a Porcupine key but not detecting:** Try increasing sensitivity:
  ```bash
  manusclaw voice wake --start --word "hey manus" --sensitivity 0.9
  ```
- **Custom wake word not supported:** Porcupine supports custom keywords via the Picovoice console. Google STT fallback supports any phrase.

### Error: `OSError: [Errno -9998] Invalid number of channels` or no audio input

**Symptom:** PyAudio fails to open the microphone.

**Root Cause:** The default audio input device is not accessible or doesn't support the required format.

**Solutions:**

```bash
# List available audio devices
python3 -c "import pyaudio; p = pyaudio.PyAudio(); [print(f'{i}: {p.get_device_info_by_index(i)[\"name\"]}') for i in range(p.get_device_count())]"
```

Then specify the device index:
```bash
export MANUSCLAW_AUDIO_INPUT_DEVICE=2  # Use the index from the list above
manusclaw voice talk --start
```

On **Termux/Android**, voice features are limited — microphone access may be restricted.

On **WSL2**, audio devices require USB/pulseaudio passthrough (see the [WSL2 Guide](platforms/wsl.md)).

### Error: TTS not producing audio output

**Symptom:** The agent responds but no audio plays through speakers.

**Solutions:**

1. **Check which TTS provider is active:**
   ```bash
   echo $ELEVENLABS_API_KEY     # Best quality
   echo $OPENAI_API_KEY         # Good quality, uses tts-1
   ```

2. **If no TTS key is set, fallback to system TTS:**
   ```bash
   pip install pyttsx3
   manusclaw voice talk --start --tts-engine system
   ```

3. **Test audio output device:**
   ```bash
   python3 -c "import pyaudio; p = pyaudio.PyAudio(); print(p.get_default_output_device_info())"
   ```

---

## SSH Gateway Issues (v5)

### Error: SSH connection refused on port 2222

**Symptom:** `ssh admin@your-server:2222` fails with "Connection refused."

**Diagnosis:**

```bash
# Check if the SSH gateway is running
sudo systemctl status manusclaw-ssh

# Check if the port is listening
ss -tlnp | grep 2222

# Check the config
echo $MANUSCLAW_SSH_ENABLED
echo $MANUSCLAW_SSH_PORT
```

**Solutions:**

1. **Enable SSH in config:**
   ```bash
   export MANUSCLAW_SSH_ENABLED=true
   export MANUSCLAW_SSH_PORT=2222
   ```

2. **Start the SSH gateway:**
   ```bash
   manusclaw-ssh start
   ```

3. **Open the firewall port:**
   ```bash
   sudo ufw allow 2222/tcp
   ```

4. **Check Docker port mapping** (if running in Docker):
   ```bash
   # Ensure port 2222 is mapped in docker-compose.yml
   docker compose logs manusclaw | grep ssh
   ```

### Error: SSH public key auth fails

**Symptom:** SSH connection is refused with "Permission denied (publickey)."

**Root Cause:** Your public key is not in the authorized keys file.

**Solution:**

```bash
# Add your public key
cat ~/.ssh/id_ed25519.pub >> ~/.manusclaw/ssh/authorized_keys

# Or configure the path
export MANUSCLAW_SSH_AUTH_KEYS=~/.ssh/authorized_keys
```

### Error: SSH command rejected — "unknown command"

**Symptom:** After connecting via SSH, your command is rejected.

**Root Cause:** The SSH gateway has a strict command whitelist. Only these commands are allowed: `status`, `restart`, `logs`, `agent <prompt>`, `channels list`, `cron list`, `help`, `exit`.

**Solution:** Use only whitelisted commands. If you need shell access, SSH directly to the host machine instead of using the ManusClaw SSH gateway.

---

## Channel Adapter Issues (v5)

### Error: Telegram channel not receiving messages

**Symptom:** The Telegram bot is running but doesn't respond to messages.

**Diagnosis:**

```bash
# Check if the bot token is valid
curl "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/getMe"
```

**Solutions:**

1. **Verify the bot token** — it should return your bot's username and ID
2. **Check webhook configuration** — if running behind Nginx, ensure the webhook URL is correct:
   ```bash
   curl "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/getWebhookInfo"
   ```
3. **Restart the channel:**
   ```bash
   manusclaw-channels start telegram
   ```

### Error: Matrix channel fails to sync

**Symptom:** Matrix adapter starts but doesn't receive new messages.

**Root Cause:** Common causes include:
- Invalid access token
- The bot account is already running elsewhere (conflicting sessions)
- Homeserver rate limiting

**Solutions:**

1. **Verify your access token:**
   ```bash
   curl -H "Authorization: Bearer $MATRIX_ACCESS_TOKEN" "$MATRIX_HOMESERVER/_matrix/client/v3/account/whoami"
   ```

2. **Regenerate the token:**
   - Log in to your bot account at `https://app.element.io`
   - Settings → Help & About → Advanced → Access Token

3. **Check homeserver logs for rate limiting:**
   - Some homeservers (especially matrix.org) have strict rate limits for `/sync` requests

### Error: Discord bot not connecting

**Symptom:** Discord channel fails with gateway connection error.

**Diagnosis:**

```bash
# Check bot token validity
curl -H "Authorization: Bot $DISCORD_BOT_TOKEN" https://discord.com/api/v10/users/@me
```

**Solutions:**

1. **Verify the bot is added to the guild** — check the Discord Developer Portal
2. **Ensure correct intents are enabled:**
   - Go to Discord Developer Portal → Bot → Privileged Gateway Intents
   - Enable "Message Content Intent" and "Server Members Intent"
3. **Restart the channel:**
   ```bash
   manusclaw-channels start discord
   ```

### Error: IRC channel connection drops

**Symptom:** IRC adapter connects but gets disconnected after a few minutes.

**Root Cause:** IRC servers enforce PING/PONG keepalive. If the adapter doesn't respond in time, the server drops the connection.

**Solutions:**

1. **Check your connection stability**
2. **Try a different IRC server or port:**
   ```bash
   export IRC_SERVER=irc.libera.chat
   export IRC_PORT=6697  # TLS port
   ```
3. **Reconnect:**
   ```bash
   manusclaw-channels start irc
   ```

---

## Webhook Issues (v5)

### Error: Webhook signature verification failed (403)

**Symptom:** External services send POST requests to your webhook endpoint but receive `403 Forbidden`.

**Root Cause:** The webhook payload is missing or has an invalid HMAC signature in the `X-Signature` header.

**Solutions:**

1. **Verify the secret matches:**
   ```bash
   manusclaw-webhook list
   # Check the secret for your webhook ID
   ```

2. **Generate a test signature:**
   ```bash
   manusclaw-webhook sign --id your-webhook-id \
     --payload '{"test": "data"}'
   ```

3. **Ensure the sender includes the signature header:**
   - GitHub: Automatically includes `X-Hub-Signature-256` when you set the secret
   - Stripe: Includes `Stripe-Signature` header
   - Custom: Must include `X-Signature` header with `sha256=<hmac_hex>`

4. **Check Nginx is forwarding signature headers:**
   ```nginx
   proxy_set_header X-Signature $http_x_signature;
   proxy_set_header X-Webhook-Signature $http_x_webhook_signature;
   ```

### Error: Webhook not triggering agent

**Symptom:** Webhook receives the request but no agent is spawned.

**Diagnosis:**

```bash
# Check webhook logs
journalctl -u manusclaw -n 50 | grep webhook
```

**Solutions:**

1. **Verify the prompt template is valid:**
   ```bash
   manusclaw-webhook list
   ```
2. **Test manually:**
   ```bash
   curl -X POST http://localhost:8765/webhooks/your-webhook-id \
     -H "Content-Type: application/json" \
     -H "X-Signature: $(manusclaw-webhook sign --id your-webhook-id --payload '{"test": true}' | tail -1)" \
     -d '{"test": true}'
   ```

---

## Model Failover Issues (v5)

### Error: Model failover not triggering

**Symptom:** When the primary model fails, ManusClaw returns an error instead of falling back to the next provider.

**Diagnosis:**

```bash
# Check if failover profiles are configured
cat ~/.manusclaw/config.yaml | grep -A10 model_profiles
```

**Solutions:**

1. **Ensure failover profiles are defined in config.yaml:**
   ```yaml
   model_profiles:
     default:
       - provider: openai
         model: gpt-4o
         priority: 1
       - provider: anthropic
         model: claude-sonnet-4-20250514
         priority: 2
   ```

2. **Check that provider API keys are set** — failover can't work if the fallback provider has no credentials

3. **Check cooldown settings** — if all providers are in cooldown, you'll get a NullResponse:
   ```bash
   # Check logs for cooldown messages
   journalctl -u manusclaw | grep cooldown
   ```

4. **Reset cooldown manually** (restart the server):
   ```bash
   sudo systemctl restart manusclaw
   ```

### Error: All models returning NullResponse

**Symptom:** Every request returns a NullResponse or empty result.

**Root Cause:** All configured providers are in cooldown (failed too many times).

**Solutions:**

1. **Check your API keys** — they may have expired or been revoked
2. **Check your network** — you may not be able to reach the providers
3. **Check rate limits** — all keys may be exhausted
4. **Restart to reset cooldowns:**
   ```bash
   sudo systemctl restart manusclaw
   ```

---

## Credential Pool Issues (v5)

### Error: Credential pool exhaustion

**Symptom:** All API keys in the pool are rate-limited and requests fail.

**Diagnosis:**

```bash
# Check how many keys are configured
env | grep OPENAI_API_KEY
# Should show OPENAI_API_KEY_1, OPENAI_API_KEY_2, etc.
```

**Solutions:**

1. **Add more keys to the pool:**
   ```bash
   cat >> ~/.manusclaw/.env << 'EOF'
   OPENAI_API_KEY_4=sk-proj-key-four
   OPENAI_API_KEY_5=sk-proj-key-five
   EOF
   ```

2. **Upgrade your API plan** — higher tiers have higher rate limits

3. **Use model failover** to distribute load across multiple providers

4. **Check which keys are in cooldown:**
   ```bash
   journalctl -u manusclaw | grep "cooldown\|rate.limit"
   ```

---

## Gmail Pub/Sub Issues (v5)

### Error: Gmail watcher not receiving emails

**Symptom:** The Gmail Pub/Sub integration is enabled but no emails trigger agent actions.

**Diagnosis:**

1. **Check Google Cloud credentials:**
   ```bash
   echo $GOOGLE_APPLICATION_CREDENTIALS
   cat $GOOGLE_APPLICATION_CREDENTIALS | python3 -c "import json,sys; print(json.load(sys.stdin).get('type'))"
   # Should print: "service_account"
   ```

2. **Check Pub/Sub topic:**
   ```bash
   gcloud pubsub topics list | grep manusclaw
   ```

3. **Check subscription push endpoint:**
   ```bash
   gcloud pubsub subscriptions describe manusclaw-gmail-sub \
     --format='value(pushConfig.pushEndpoint)'
   ```

**Solutions:**

1. **Ensure the Pub/Sub push endpoint points to your server:**
   ```bash
   gcloud pubsub subscriptions update manusclaw-gmail-sub \
     --push-endpoint https://your-server:8765/webhooks/gmail-push
   ```

2. **Verify the Gmail watch topic is configured:**
   ```bash
   export GMAIL_WATCH_TOPIC_NAME=projects/your-project/topics/manusclaw-gmail-topic
   ```

3. **Check that the service account has Gmail API permissions:**
   - Go to Google Cloud Console → IAM & Admin → Service Accounts
   - Ensure the service account has the "Gmail API Watcher" role

4. **Test the webhook endpoint directly:**
   ```bash
   curl -k https://your-server:8765/health
   ```

### Error: Gmail auto-reply not working

**Symptom:** Emails are received but automatic replies are not sent.

**Solution:**

```bash
# Ensure auto-reply is enabled
export GMAIL_AUTO_REPLY=true

# Check that the Gmail API scopes include send permissions
# The service account needs: gmail.compose, gmail.send
```

---

## Permission Denied Errors

### Error: `Permission denied: '/usr/lib/python3/...'` or similar

**Symptom:** pip install fails with permission errors when trying to write to system directories.

**Root Cause:** You're trying to install packages to the system Python, which requires root access. This is generally not recommended.

**Solutions:**

1. **Use a virtual environment (recommended):**
   ```bash
   python3 -m venv ~/manusclaw-env
   source ~/manusclaw-env/bin/activate
   pip install manusclaw
   ```

2. **Use `--user` flag:**
   ```bash
   pip install --user manusclaw
   ```
   Then ensure `~/.local/bin` is in your PATH.

3. **Use sudo (not recommended):**
   ```bash
   sudo pip install manusclaw
   ```
   This can break your system Python. Only use as a last resort.

### Error: `Permission denied` when writing to workspace

**Symptom:** ManusClaw can read files but cannot write to the workspace directory.

**Diagnosis:**
```bash
# Check directory permissions
ls -la workspace/
stat workspace/
```

**Solution:**
```bash
# Fix permissions on the workspace
chmod 755 workspace/
chown -R $USER:$USER workspace/
```

### Error: `Permission denied` on config files

```bash
# Fix permissions on config directory
chmod 755 ~/.manusclaw/
chmod 644 ~/.manusclaw/config.toml
chmod 600 ~/.manusclaw/.env  # .env should be more restrictive
chown -R $USER:$USER ~/.manusclaw/
```

---

## Memory / Database Errors

### Error: `MemoryError` or system runs out of RAM

**Symptom:** ManusClaw crashes or the system becomes unresponsive when processing large files or long conversations.

**Diagnosis:**
```bash
# Check memory usage
free -h

# Monitor in real-time
watch -n 1 free -h
```

**Solutions:**

1. **Reduce the token budget:**
   ```toml
   [token_budget]
   max_input_tokens = 32000
   max_total_tokens = 50000
   auto_summarize = true
   ```

2. **Use a smaller model** (especially for Ollama)

3. **Clear conversation history regularly:**
   ```
   > /clear
   ```

4. **Reduce MEMORY.md size:**
   ```
   > /memory --clear
   ```

5. **Add swap space (Linux):**
   ```bash
   sudo fallocate -l 4G /swapfile
   sudo chmod 600 /swapfile
   sudo mkswap /swapfile
   sudo swapon /swapfile
   echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
   ```

### Error: Corrupted MEMORY.md or USER.md

**Symptom:** ManusClaw behaves erratically or crashes when loading memory files.

**Solution:**

```bash
# Back up the corrupted file
cp ~/.manusclaw/MEMORY.md ~/.manusclaw/MEMORY.md.backup

# Reset memory
# Option 1: From within ManusClaw
> /memory --clear

# Option 2: Manually delete the file
rm ~/.manusclaw/MEMORY.md

# ManusClaw will create a fresh MEMORY.md on next start
```

---

## Network / Firewall Issues

### Error: `ConnectionError: Failed to establish a new connection`

**Symptom:** ManusClaw cannot reach the LLM provider's API.

**Diagnosis:**
```bash
# Test basic connectivity
curl -I https://api.openai.com

# Test with your API key
curl https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY"

# Check DNS resolution
nslookup api.openai.com

# Check for proxy settings
echo $HTTP_PROXY
echo $HTTPS_PROXY
```

**Solutions:**

1. **Check your internet connection** (obvious, but often overlooked)

2. **Check firewall rules:**
   ```bash
   # Linux (ufw)
   sudo ufw status

   # Allow outbound HTTPS
   sudo ufw allow out 443/tcp
   ```

3. **Check if you're behind a proxy:**
   ```bash
   export HTTPS_PROXY="http://proxy.example.com:8080"
   export HTTP_PROXY="http://proxy.example.com:8080"
   ```

4. **Check if you're in a restricted network** (corporate firewall, China, etc.):
   - You may need to use a VPN
   - Some countries block API endpoints — consider using OpenRouter as an alternative

5. **DNS issues:**
   ```bash
   # Try using Google's DNS
   echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
   ```

### Error: SSL certificate verification failed

**Symptom:** `SSLError: CERTIFICATE_VERIFY_FAILED`

**Solution:**

```bash
# Update CA certificates (Linux)
sudo update-ca-certificates

# macOS
/Applications/Python\ 3.11/Install\ Certificates.command

# Temporary workaround (NOT recommended for production)
export PYTHONHTTPSVERIFY=0
# Or in Python: import ssl; ssl._create_default_https_context = ssl._create_unverified_context
```

---

## Windows-Specific Issues

### Error: `'manusclaw' is not recognized as an internal or external command`

**Root Cause:** The Python Scripts directory is not in your Windows PATH.

**Solution:**

1. Find your Python installation path:
   ```powershell
   python -c "import sys; print(sys.executable)"
   ```

2. Add the `Scripts` directory to your PATH. The typical path is:
   ```
   C:\Users\YourName\AppData\Local\Programs\Python\Python311\Scripts\
   ```

3. To add it to PATH:
   - Open "Edit the system environment variables" from the Start menu
   - Click "Environment Variables"
   - Under "User variables", select "Path" and click "Edit"
   - Click "New" and paste the Scripts path
   - Click OK and restart your terminal

### Error: `UnicodeDecodeError` on Windows

**Symptom:** Errors related to character encoding when reading files.

**Solution:**

```powershell
# Set the default encoding
$env:PYTHONUTF8 = "1"

# Or add to your PowerShell profile
echo '$env:PYTHONUTF8 = "1"' >> $PROFILE
```

### Error: Long path names on Windows

**Symptom:** `FileNotFoundError` even though the file exists, or errors about paths being too long.

**Solution:**

Enable long path support in Windows:

1. Open Registry Editor (`regedit`)
2. Navigate to `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem`
3. Set `LongPathsEnabled` to `1`
4. Restart your computer

Or via PowerShell (as Administrator):

```powershell
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" `
  -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
```

### Error: Antivirus blocking Python/pip

**Symptom:** Installation fails or Python scripts can't run because Windows Defender or another antivirus quarantines files.

**Solution:**

1. Add Python and your project directories to the antivirus exclusion list
2. In Windows Security: Settings → Exclusions → Add or remove exclusions
3. Add exclusions for:
   - `C:\Users\YourName\AppData\Local\Programs\Python\`
   - `C:\Users\YourName\AppData\Local\pip\`
   - Your workspace directory

---

## Termux-Specific Issues

### Error: `CCompiler._compile error` during pip install

**Symptom:** Building C extensions fails in Termux.

**Solution:**

```bash
# Install the clang compiler
pkg install clang

# Install other build dependencies
pkg install python-dev libffi openssl

# Retry the installation
pip install manusclaw
```

### Error: `No space left on device` in Termux

**Symptom:** Termux runs out of storage space, especially on devices with limited storage.

**Solution:**

```bash
# Check disk usage
df -h

# Clean pip cache
pip cache purge

# Clean Termux package cache
pkg clean

# Move Termux to shared storage (if possible)
# Use Termux's internal storage setup:
termux-setup-storage

# Or install on an SD card (requires Android 6+ with adopted storage)
```

### Error: Process killed by Android OOM killer

**Symptom:** ManusClaw or Ollama is killed unexpectedly, especially in the background.

**Solution:**

```bash
# Prevent Android from killing background processes
termux-wake-lock

# Reduce memory usage in config
# Use a smaller model or reduce token limits

# Run in foreground (not background) when possible
```

### Error: Playwright not available

**Root Cause:** Termux cannot run desktop browsers. This is a hard limitation.

**Workaround:** Use DuckDuckGo search instead of browser-based browsing. Configure:

```toml
[search]
engine = "duckduckgo"
```

### Error: Voice features not working on Termux

**Root Cause:** Termux/Android has limited audio device access. PyAudio may not be able to access the microphone or speakers.

**Workaround:** Voice features (wake word, talk mode) are not fully supported on Termux. Use text-based interaction instead. If you need voice, use Termux's SSH to connect to a ManusClaw instance running on a desktop machine.

---

## Docker-Specific Issues

### Error: `Cannot connect to the Docker daemon`

**Symptom:** Docker commands fail with a connection error.

**Solution:**

```bash
# Start Docker
sudo systemctl start docker

# Add your user to the docker group
sudo usermod -aG docker $USER

# Log out and back in, or:
newgrp docker

# Verify
docker ps
```

### Error: ManusClaw container can't connect to Ollama container

**Symptom:** ManusClaw in Docker can't reach Ollama at `localhost:11434`.

**Root Cause:** In Docker, `localhost` inside a container refers to the container itself, not the host or other containers.

**Solution:**

1. **Use Docker networking (recommended):**
   ```bash
   # Create a network
   docker network create manusclaw-net

   # Run both containers on the same network
   docker run --network manusclaw-net -e OLLAMA_BASE_URL=http://ollama:11434 ...
   ```

2. **Use host.docker.internal (Docker Desktop):**
   ```bash
   docker run -e OLLAMA_BASE_URL=http://host.docker.internal:11434 ...
   ```

3. **Use the host network mode:**
   ```bash
   docker run --network host ...
   ```
   This removes network isolation, so the container shares the host's network stack.

### Error: Data lost after container restart

**Root Cause:** You're not using volume mounts. Without volumes, all data inside the container is ephemeral.

**Solution:**

Always use volume mounts for persistent data:

```bash
docker run -v manusclaw-config:/root/.manusclaw \
           -v manusclaw-workspace:/app/workspace \
           manusclaw
```

### Error: Container exits immediately

**Diagnosis:**

```bash
# Check the exit code
docker ps -a

# View the logs
docker logs manusclaw-server

# Run interactively to see the error
docker run -it manusclaw manusclaw-server --host 0.0.0.0
```

Common causes:
- Missing API keys (add `-e OPENAI_API_KEY=...`)
- Invalid config.toml (check the mounted config file)
- Port already in use (change the port mapping)

### Error: `version` key is deprecated in docker-compose.yml

**Symptom:** Docker Compose warns about the `version: "3.8"` key.

**Solution:** Remove the `version:` key. Modern Docker Compose (v2+) ignores it and uses profiles instead. See the [Deployment Guide](deployment.md) for updated compose files.

---

## Rate Limiting Issues

### Error: `RateLimitError: Rate limit reached for default`

**Symptom:** Requests are throttled or rejected by the LLM provider.

**Solutions:**

1. **Add more API keys to the credential pool:**
   ```env
   OPENAI_API_KEY_1=sk-key1
   OPENAI_API_KEY_2=sk-key2
   ```
   See [Configuration Guide](configuration.md#credential-pool) for details.

2. **Reduce request frequency:**
   - Don't send too many requests in quick succession
   - Use single-shot mode instead of rapid interactive queries
   - Combine multiple small questions into one larger query

3. **Switch to a provider with higher limits:**
   - OpenAI's Tier 1: 500 RPM
   - OpenAI's Tier 2: 5,000 RPM
   - Anthropic: Varies by plan
   - Google: 60 RPM (free), 2,000 RPM (paid)

4. **Enable model failover** (v5) — automatically routes to another provider when one is rate-limited:
   ```yaml
   model_profiles:
     default:
       - provider: openai
         model: gpt-4o
         priority: 1
       - provider: groq
         model: llama-3.3-70b-versatile
         priority: 2
   ```

5. **Increase the retry delay:**
   ```toml
   [llm]
   retries = 5
   
   # Or in the provider section
   [llm.openai]
   timeout = 120
   ```

6. **Use a different model** with higher rate limits (e.g., `gpt-4o-mini` instead of `gpt-4o`)

---

## Token Budget Exhausted

### Error: `Token budget exhausted` or conversation suddenly becomes very limited

**Symptom:** ManusClaw warns that the token budget is running low, or responses become truncated.

**Root Cause:** The conversation history has grown too large, consuming most of the model's context window.

**Solutions:**

1. **Clear the conversation:**
   ```
   > /clear
   ```

2. **Increase the token budget** (if your model supports a larger context):
   ```toml
   [token_budget]
   max_input_tokens = 200000
   max_total_tokens = 300000
   ```

3. **Enable auto-summarize:**
   ```toml
   [token_budget]
   auto_summarize = true
   warning_threshold = 0.7
   ```

4. **Switch to a model with a larger context window:**
   - GPT-4o: 128K tokens
   - Claude 3.5 Sonnet: 200K tokens
   - Gemini 1.5 Pro: 1M tokens

5. **Reduce the max_tokens per response:**
   ```toml
   [llm]
   max_tokens = 2048  # Reduce from 4096
   ```

6. **Check token usage:**
   ```
   > /token-usage
   ```

---

## How to Reset / Clean Install

If nothing else works, a clean installation can resolve persistent issues. This section provides step-by-step instructions for completely removing and reinstalling ManusClaw.

### Step 1: Uninstall ManusClaw

```bash
pip uninstall manusclaw -y
```

### Step 2: Remove configuration files

```bash
# Remove the entire config directory
rm -rf ~/.manusclaw

# Or selectively remove:
rm ~/.manusclaw/config.toml
rm ~/.manusclaw/.env
rm ~/.manusclaw/MEMORY.md
rm ~/.manusclaw/USER.md
rm -rf ~/.manusclaw/sessions
rm -rf ~/.manusclaw/tasks
rm -rf ~/.manusclaw/skills
rm -rf ~/.manusclaw/profiles
rm -rf ~/.manusclaw/ssh
rm -f ~/.manusclaw/cron.yaml
```

### Step 3: Remove workspace data (if needed)

```bash
# ⚠️ This deletes all your project files that were in the workspace
rm -rf workspace/
```

### Step 4: Remove pip cache

```bash
pip cache purge
```

### Step 5: Remove virtual environment (if using one)

```bash
rm -rf ~/manusclaw-env
```

### Step 6: Remove Playwright browsers

```bash
# Remove installed browsers
playwright uninstall --all

# Remove the Playwright cache
rm -rf ~/Library/Caches/ms-playwright    # macOS
rm -rf ~/.cache/ms-playwright            # Linux
```

### Step 7: Fresh install

```bash
# Create a new virtual environment
python3 -m venv ~/manusclaw-env
source ~/manusclaw-env/bin/activate

# Install ManusClaw
pip install --upgrade pip
pip install manusclaw

# Install Playwright browsers
playwright install --with-deps chromium

# Verify
manusclaw --version
```

### Step 8: Reconfigure

```bash
# Set up API keys
echo 'export OPENAI_API_KEY="sk-your-key"' >> ~/.bashrc
source ~/.bashrc

# Or create a new .env file
mkdir -p ~/.manusclaw
cat > ~/.manusclaw/.env << 'EOF'
OPENAI_API_KEY=sk-your-key
EOF
chmod 600 ~/.manusclaw/.env

# Test
manusclaw "Hello, world!"
```

### Nuclear option: Remove everything related to Python

**⚠️ WARNING: This removes ALL Python packages, not just ManusClaw. Only use as an absolute last resort.**

```bash
# Remove all pip packages
pip freeze | xargs pip uninstall -y

# Remove Python cache
find ~ -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null
find ~ -type d -name ".pytest_cache" -exec rm -rf {} + 2>/dev/null
find ~ -type f -name "*.pyc" -delete 2>/dev/null

# Remove the virtual environment
rm -rf ~/manusclaw-env

# Then reinstall from scratch
python3 -m venv ~/manusclaw-env
source ~/manusclaw-env/bin/activate
pip install manusclaw
```
