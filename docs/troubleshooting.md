# Troubleshooting Guide — ManusClaw v5.1.0

This guide covers the most common issues you might encounter when installing, configuring, or running ManusClaw v5.1.0. Each problem includes a detailed explanation of the root cause and step-by-step solutions. If your issue isn't covered here, please open a GitHub issue with the details of your environment and the error message you're seeing.

---

## Table of Contents

- [Common Installation Errors](#common-installation-errors)
- [Python Version Issues](#python-version-issues)
- [pip / Dependency Conflicts](#pip--dependency-conflicts)
- [API Key Configuration Problems](#api-key-configuration-problems)
- [Ollama Connection Issues](#ollama-connection-issues)
- [Playwright Browser Issues](#playwright-browser-issues)
- [Voice & Audio Issues](#voice--audio-issues)
- [SSH Gateway Issues](#ssh-gateway-issues)
- [Channel Adapter Issues](#channel-adapter-issues)
- [Webhook Issues](#webhook-issues)
- [Model Failover Issues](#model-failover-issues)
- [Credential Pool Issues](#credential-pool-issues)
- [Security & RBAC Issues (v5.1)](#security--rbac-issues-v51)
- [Hooks Issues (v5.1)](#hooks-issues-v51)
- [Secrets Management Issues (v5.1)](#secrets-management-issues-v51)
- [Observability Issues (v5.1)](#observability-issues-v51)
- [File Store Issues (v5.1)](#file-store-issues-v51)
- [Git Provider Issues (v5.1)](#git-provider-issues-v51)
- [Parallel Executor Issues (v5.1)](#parallel-executor-issues-v51)
- [Migration Issues (v5.1)](#migration-issues-v51)
- [Context Management Issues (v5.1)](#context-management-issues-v51)
- [Permission Denied Errors](#permission-denied-errors)
- [Memory / Database Errors](#memory--database-errors)
- [Network / Firewall Issues](#network--firewall-issues)
- [Windows-Specific Issues](#windows-specific-issues)
- [Termux-Specific Issues](#termux-specific-issues)
- [Docker-Specific Issues](#docker-specific-issues)
- [Kubernetes Issues (v5.1)](#kubernetes-issues-v51)
- [Rate Limiting Issues](#rate-limiting-issues)
- [Token Budget Exhausted](#token-budget-exhausted)
- [How to Reset / Clean Install](#how-to-reset--clean-install)

---

## Common Installation Errors

### Error: `command not found: manusclaw`

**Cause:** ManusClaw is not in your PATH, or the installation failed.

```bash
# Check if manusclaw is installed
pip show manusclaw

# If not installed, install it
pip install manusclaw

# If installed but not in PATH, find the binary
pip show -f manusclaw | grep bin

# Add to PATH (adjust path as needed)
export PATH="$PATH:$(python -m site --user-base)/bin"

# Or reinstall
pip install --force-reinstall manusclaw
```

### Error: `error: externally-managed-environment`

**Cause:** Newer Linux distributions (Ubuntu 23.04+, Fedora 38+) prevent global pip installs.

```bash
# Option 1: Use a virtual environment (recommended)
python3 -m venv ~/manusclaw-env
source ~/manusclaw-env/bin/activate
pip install manusclaw

# Option 2: Use pipx
pipx install manusclaw

# Option 3: Override (not recommended)
pip install --break-system-packages manusclaw
```

### Error: `Failed building wheel for pydantic-core` / `cryptography`

**Cause:** Missing C compiler or Rust compiler for building C/Rust extensions.

```bash
# Ubuntu/Debian
sudo apt install -y build-essential python3-dev libffi-dev libssl-dev

# For pydantic-core (needs Rust)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env
pip install manusclaw

# For cryptography
sudo apt install -y libffi-dev libssl-dev cargo
pip install cryptography
pip install manusclaw
```

---

## Python Version Issues

### Error: ManusClaw requires Python 3.11+

**Cause:** Your Python version is too old.

```bash
# Check your Python version
python --version

# Install a newer Python (Ubuntu/Debian)
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.12 python3.12-venv

# Use the newer Python
python3.12 -m venv ~/manusclaw-env
source ~/manusclaw-env/bin/activate
pip install manusclaw
```

---

## pip / Dependency Conflicts

### Error: `ResolutionImpossible` or dependency conflicts

**Cause:** Conflicting package versions in your environment.

```bash
# Create a fresh virtual environment
python3 -m venv ~/mc-fresh
source ~/mc-fresh/bin/activate
pip install --upgrade pip
pip install manusclaw

# If conflicts persist, try with --no-deps and install dependencies manually
pip install manusclaw --no-deps
pip install pydantic pyyaml toml httpx rich prompt_toolkit python-dotenv aiofiles
```

---

## API Key Configuration Problems

### Error: `OPENAI_API_KEY not set` or `Authentication error`

**Cause:** API keys are not configured properly.

```bash
# Check if the key is set
echo $OPENAI_API_KEY

# Set the key temporarily
export OPENAI_API_KEY="sk-proj-your-key"

# Set permanently in .env
mkdir -p ~/.manusclaw
echo 'OPENAI_API_KEY=sk-proj-your-key' >> ~/.manusclaw/.env

# Or in shell profile
echo 'export OPENAI_API_KEY="sk-proj-your-key"' >> ~/.bashrc
source ~/.bashrc

# For v5.1 Vault integration
manusclaw secrets set openai/api_key "sk-proj-your-key"
```

### Error: `Invalid API Key`

**Cause:** The API key is incorrect, expired, or doesn't have the required permissions.

```bash
# Test your OpenAI key
curl -s https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY" | head -c 200

# Test Anthropic key
curl -s https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-20250514","max_tokens":10,"messages":[{"role":"user","content":"Hi"}]}'

# Check if using the wrong key for the provider
manusclaw config get llm.provider
```

---

## Ollama Connection Issues

### Error: `Cannot connect to Ollama at http://localhost:11434`

**Cause:** Ollama is not running or not accessible.

```bash
# Check if Ollama is running
curl http://localhost:11434/api/tags

# Start Ollama
ollama serve &

# Check Ollama models
ollama list

# If using a custom host, verify the URL
curl http://your-ollama-host:11434/api/tags

# Update config
export OLLAMA_BASE_URL=http://your-ollama-host:11434
# Or in config.toml:
# [llm.ollama]
# base_url = "http://your-ollama-host:11434"
```

### Error: Model not found in Ollama

```bash
# List available models
ollama list

# Pull the model
ollama pull llama3

# Use the exact model name from `ollama list`
manusclaw --provider ollama --model llama3 "Hello"
```

---

## Playwright Browser Issues

### Error: `playwright._impl._errors.Error: BrowserType.launch() failed`

**Cause:** Playwright browsers are not installed.

```bash
# Install Playwright browsers
playwright install chromium

# Install system dependencies (Linux)
playwright install-deps chromium

# Full install
playwright install --with-deps chromium
```

### Error: Playwright fails in Docker

```bash
# Use the Playwright Docker image
FROM mcr.microsoft.com/playwright/python:v1.40.0-jammy

# Or install deps in your Dockerfile
RUN playwright install-deps chromium
RUN playwright install chromium
```

---

## Voice & Audio Issues

### Error: `PyAudio not found` or `No Default Input Device Available`

**Cause:** PortAudio is not installed or audio devices are not accessible.

```bash
# Install PortAudio (Ubuntu/Debian)
sudo apt install -y portaudio19-dev python3-pyaudio

# Install PortAudio (macOS)
brew install portaudio

# Install PortAudio (Fedora)
sudo dnf install -y portaudio-devel

# Reinstall PyAudio
pip install --force-reinstall pyaudio
```

### Error: Wake word not detecting

```bash
# Check your Porcupine API key
echo $PICOVOICE_API_KEY

# Try with higher sensitivity
manusclaw voice wake --start --word "hey manus" --sensitivity 0.9

# Try Google STT fallback
manusclaw voice wake --start --word "hey manus" --backend google

# Check microphone permissions
python -c "import pyaudio; p=pyaudio.PyAudio(); print(p.get_device_count())"
```

### Error: TTS not working

```bash
# Check ElevenLabs API key
echo $ELEVENLABS_API_KEY

# Try system TTS fallback
manusclaw voice talk --start --tts system

# Try OpenAI TTS
manusclaw voice talk --start --tts openai
```

---

## SSH Gateway Issues

### Error: `SSH connection refused`

**Cause:** SSH gateway is not running or the port is blocked.

```bash
# Check if SSH gateway is running
ps aux | grep manusclaw-ssh

# Check if the port is open
ss -tlnp | grep 2222

# Start the SSH gateway
export MANUSCLAW_SSH_ENABLED=true
manusclaw-ssh start

# Check logs
manusclaw-ssh logs
```

### Error: `Permission denied (publickey)`

**Cause:** SSH public key authentication is not configured correctly.

```bash
# Verify the authorized_keys file
cat ~/.ssh/authorized_keys

# Set correct permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Set the auth keys path
export MANUSCLAW_SSH_AUTH_KEYS=~/.ssh/authorized_keys
```

---

## Channel Adapter Issues

### Error: `Telegram Bot token not set`

```bash
# Set the bot token
export TELEGRAM_BOT_TOKEN="your-bot-token"

# Verify the token
curl "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/getMe"
```

### Error: `Discord connection failed`

```bash
# Check the bot token
echo $DISCORD_BOT_TOKEN

# Verify the token (check for extra spaces)
echo "$DISCORD_BOT_TOKEN" | wc -c

# Check Discord bot permissions in the Discord Developer Portal
```

### Error: `LINE channel authentication failed` (v5.1)

```bash
# Verify LINE credentials
echo $LINE_CHANNEL_SECRET
echo $LINE_CHANNEL_ACCESS_TOKEN

# Test the access token
curl -H "Authorization: Bearer $LINE_CHANNEL_ACCESS_TOKEN" \
  https://api.line.me/v2/bot/info
```

---

## Webhook Issues

### Error: `HMAC signature verification failed`

**Cause:** The webhook secret is incorrect or the signature header is missing.

```bash
# Verify the webhook secret
manusclaw-webhook list

# Generate a test signature
manusclaw-webhook sign --id my-webhook --payload '{"test": true}'

# Check the signature header name (should be X-Signature)
curl -X POST http://localhost:8765/webhooks/my-webhook \
  -H "Content-Type: application/json" \
  -H "X-Signature: $(manusclaw-webhook sign --id my-webhook --payload '{"test": true}' 2>/dev/null)" \
  -d '{"test": true}'
```

---

## Model Failover Issues

### Error: `All providers in profile failed`

**Cause:** All LLM providers in the failover chain are unavailable.

```bash
# Check which providers are in cooldown
manusclaw config get model_profiles.default

# Test each provider individually
manusclaw --provider openai "test"
manusclaw --provider anthropic "test"
manusclaw --provider groq "test"

# Check API keys
echo $OPENAI_API_KEY
echo $ANTHROPIC_API_KEY
echo $GROQ_API_KEY

# Reset cooldown timers by restarting
manusclaw-server --restart
```

---

## Credential Pool Issues

### Error: `All credentials in pool are rate-limited`

**Cause:** All API keys in the credential pool have hit rate limits.

```bash
# Check which keys are rate-limited
manusclaw credentials status

# Add more keys
export OPENAI_API_KEY_4=sk-proj-another-key

# Wait for cooldown (typically 60 seconds)
sleep 60
```

---

## Security & RBAC Issues (v5.1)

### Error: `Permission denied: agent:execute`

**Cause:** The user's role doesn't have the required permission.

```bash
# Check the user's role
manusclaw security users permissions --user-id <user-id>

# Assign a higher role
manusclaw security users assign --user-id <user-id> --role user

# Or add the specific permission to the role
manusclaw security roles update --name viewer --add-permissions "agent:execute"
```

### Error: `Rate limit exceeded`

**Cause:** The user has exceeded their configured rate limit.

```bash
# Check rate limit status
manusclaw security rate-limit status

# Increase the rate limit
manusclaw config set security.rate_limiting.requests_per_minute 120

# Reset the user's rate limit
manusclaw security rate-limit reset --user-id <user-id>
```

### Error: `Input validation rejected: blocked pattern detected`

**Cause:** Input validation detected a blocked pattern in the user's message.

```bash
# Check blocked patterns
manusclaw config get security.input_validation.blocked_patterns

# Remove or adjust patterns
# Edit ~/.manusclaw/config.yaml and update security.input_validation.blocked_patterns

# Temporarily disable strict mode
manusclaw config set security.input_validation.strict_mode false
```

---

## Hooks Issues (v5.1)

### Error: `Hook 'log_request' timed out after 30s`

**Cause:** A hook script or function took too long to execute.

```bash
# Increase the hook timeout
# In config.yaml:
# hooks:
#   pre_execute:
#     - name: "log_request"
#       timeout: 60  # Increase from 30 to 60

# Check hook execution logs
manusclaw hooks list
manusclaw hooks test --name "log_request" --hook-type pre_execute
```

### Error: `Hook 'validate_input' aborted execution`

**Cause:** A pre-execute hook returned a non-zero exit code or `False`.

```bash
# Check the hook's on_failure policy
manusclaw config get hooks.pre_execute

# Temporarily disable the hook
manusclaw hooks disable --name "validate_input"

# Change on_failure from "abort" to "warn"
# In config.yaml, set on_failure: "warn" for the hook
```

---

## Secrets Management Issues (v5.1)

### Error: `Cannot connect to Vault at http://localhost:8200`

**Cause:** HashiCorp Vault is not running or the address is incorrect.

```bash
# Check Vault status
vault status

# Start Vault in dev mode (for testing)
vault server -dev &

# Verify the connection
curl http://localhost:8200/v1/sys/health

# Check VAULT_ADDR environment variable
echo $VAULT_ADDR
```

### Error: `Vault authentication failed`

**Cause:** The Vault token or authentication method is not configured correctly.

```bash
# Check the Vault token
echo $VAULT_TOKEN

# Test Vault authentication
vault login $VAULT_TOKEN

# For AppRole authentication
export VAULT_ROLE_ID=your-role-id
export VAULT_SECRET_ID=your-secret-id
vault write auth/approle/login role_id="$VAULT_ROLE_ID" secret_id="$VAULT_SECRET_ID"
```

### Error: `Secret not found: openai/api_key`

**Cause:** The secret path doesn't exist in the secrets backend.

```bash
# List available secrets
manusclaw secrets list

# Store the secret
manusclaw secrets set openai/api_key "sk-proj-xxx"

# Check if using the correct backend
manusclaw config get secrets.backend
```

---

## Observability Issues (v5.1)

### Error: `OpenTelemetry export failed: connection refused`

**Cause:** The OTLP collector is not running or the endpoint is incorrect.

```bash
# Check the OTLP endpoint
echo $OTEL_EXPORTER_OTLP_ENDPOINT

# Verify the collector is running
curl http://localhost:4318/v1/traces

# For Docker deployments, ensure the collector is started:
docker compose --profile observability up -d otel-collector
```

### Error: `Prometheus metrics not available`

**Cause:** Metrics endpoint is not enabled or the port is blocked.

```bash
# Check if metrics are enabled
manusclaw config get observability.metrics.enabled

# Enable metrics
manusclaw config set observability.metrics.enabled true

# Check the metrics port
curl http://localhost:9090/metrics

# For Docker, ensure port 9090 is exposed
```

---

## File Store Issues (v5.1)

### Error: `S3 upload failed: Access Denied`

**Cause:** AWS credentials don't have permission to write to the S3 bucket.

```bash
# Check AWS credentials
aws sts get-caller-identity

# Test S3 access
aws s3 ls s3://your-bucket/

# Check bucket permissions
aws s3api get-bucket-policy --bucket your-bucket

# Verify IAM permissions include s3:PutObject, s3:GetObject
```

### Error: `File store: local path not writable`

**Cause:** The ManusClaw process doesn't have write permission to the file store directory.

```bash
# Check directory permissions
ls -la ~/.manusclaw/files

# Fix permissions
mkdir -p ~/.manusclaw/files
chmod 755 ~/.manusclaw/files

# Or change the file store path
manusclaw config set file_store.local.base_path /path/to/writable/dir
```

---

## Git Provider Issues (v5.1)

### Error: `GitHub API: rate limit exceeded`

**Cause:** GitHub API rate limit (5000 requests/hour for authenticated users) has been exceeded.

```bash
# Check rate limit status
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/rate_limit | python -m json.tool

# Wait for rate limit reset (check reset timestamp)
# Or use a different token
```

### Error: `GitLab authentication failed`

**Cause:** The GitLab token is invalid or doesn't have the required scope.

```bash
# Test the GitLab token
curl -s -H "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  https://gitlab.com/api/v4/user | python -m json.tool

# Verify token scopes in GitLab Settings → Access Tokens
```

---

## Parallel Executor Issues (v5.1)

### Error: `Parallel executor: maximum workers exceeded`

**Cause:** Too many concurrent tasks for the configured worker pool.

```bash
# Check current worker pool status
manusclaw-parallel status

# Increase max workers
manusclaw config set parallel_executor.workers.max_workers 8

# Cancel stuck tasks
manusclaw-parallel cancel --all
```

### Error: `Dependency cycle detected`

**Cause:** Task dependencies form a circular reference.

```bash
# Review task dependencies
manusclaw-parallel status --show-deps

# Break the cycle by removing a dependency
# Re-run tasks with corrected dependencies
```

### Error: `Ray initialization failed`

**Cause:** Ray cluster is not available or incompatible version.

```bash
# Check Ray status
ray status

# Start Ray locally
ray start --head

# Or switch to threaded mode
manusclaw config set parallel_executor.mode threaded
```

---

## Migration Issues (v5.1)

### Error: `Migration from 5.0.0 to 5.1.0 failed`

**Cause:** Migration encountered an error, likely due to corrupted config or missing files.

```bash
# Check migration status
manusclaw-migrate status

# Check the backup directory
ls ~/.manusclaw/migrations/backups/

# Rollback the failed migration
manusclaw-migrate rollback

# Try manual migration
manusclaw-migrate --from 5.0.0 --to 5.1.0 --verbose
```

### Error: `Migration state file corrupted`

```bash
# Remove the corrupted state file
rm ~/.manusclaw/migrations/.migration-state

# Re-run migrations
manusclaw-migrate run
```

---

## Context Management Issues (v5.1)

### Error: `Context window exceeded`

**Cause:** The conversation has grown larger than the configured context window.

```bash
# Check context usage
manusclaw context status

# Manually trigger compression
manusclaw context compress

# Enable auto-compression
manusclaw config set context.compression.enabled true

# Increase context window size
manusclaw config set context.window.max_tokens 200000

# Switch to hybrid compression strategy
manusclaw config set context.compression.strategy hybrid
```

### Error: `Context summary quality is poor`

**Cause:** The summarization model is not powerful enough for good summaries.

```bash
# Specify a better model for summarization
manusclaw config set context.compression.summarization.model gpt-4o

# Increase the summary token budget
manusclaw config set context.compression.summarization.max_summary_tokens 4000
```

---

## Permission Denied Errors

### Error: `Permission denied: ~/.manusclaw/`

**Cause:** File ownership or permissions issue.

```bash
# Check ownership
ls -la ~ | grep manusclaw

# Fix ownership
sudo chown -R $(whoami) ~/.manusclaw

# Fix permissions
chmod 700 ~/.manusclaw
chmod 600 ~/.manusclaw/.env
```

---

## Memory / Database Errors

### Error: `Session storage error` or `Database locked`

**Cause:** Corrupted session database or concurrent access issue.

```bash
# Check database file
ls -la ~/.manusclaw/sessions/

# Remove corrupted sessions
rm -rf ~/.manusclaw/sessions/*.db

# Restart ManusClaw
manusclaw-server --restart
```

---

## Network / Firewall Issues

### Error: `Connection timeout` or `Network is unreachable`

**Cause:** Firewall blocking outgoing connections.

```bash
# Test connectivity to OpenAI
curl -s https://api.openai.com/v1/models -H "Authorization: Bearer $OPENAI_API_KEY" | head -c 100

# Test connectivity to Anthropic
curl -s https://api.anthropic.com/v1/messages -H "x-api-key: $ANTHROPIC_API_KEY" | head -c 100

# Check DNS resolution
nslookup api.openai.com

# Check firewall rules (Linux)
sudo iptables -L OUTPUT -n

# Check proxy settings
echo $HTTP_PROXY
echo $HTTPS_PROXY
```

---

## Windows-Specific Issues

### Error: `Windows is not supported for voice features`

Voice features require WSL2 or a Linux/macOS environment. See the [WSL2 Guide](platforms/wsl.md) for setup instructions.

### Error: `Long path names cause issues`

```bash
# Enable long paths in Windows (run as admin)
reg add "HKLM\SYSTEM\CurrentControlSet\Control\FileSystem" /v LongPathsEnabled /t REG_DWORD /d 1 /f

# Use WSL2 instead (recommended)
```

---

## Termux-Specific Issues

### Error: `Build failed for cryptography` or `pydantic-core`

```bash
# Install Rust compiler
pkg install rust

# Install additional build deps
pkg install libffi openssl

# Retry installation
pip install manusclaw
```

### Error: `PortAudio not available`

```bash
pkg install portaudio
pip install --force-reinstall pyaudio
```

---

## Docker-Specific Issues

### Error: `Docker: Cannot connect to the Docker daemon`

```bash
# Start Docker
sudo systemctl start docker

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Test Docker
docker run hello-world
```

### Error: `Docker: permission denied while trying to connect`

```bash
# Use sudo or add your user to the docker group
sudo usermod -aG docker $USER
newgrp docker
```

### Error: `Container keeps restarting`

```bash
# Check container logs
docker logs manusclaw-server

# Check health check
docker inspect manusclaw-server | grep -A5 Health

# Run interactively for debugging
docker run -it --rm \
  -e OPENAI_API_KEY=sk-proj-xxx \
  manusclaw/manusclaw:5.1.0 \
  manusclaw "test"
```

---

## Kubernetes Issues (v5.1)

### Error: `CrashLoopBackOff`

**Cause:** The container is crashing on startup, often due to missing configuration.

```bash
# Check pod logs
kubectl logs -n manusclaw deployment/manusclaw

# Check pod events
kubectl describe pod -n manusclaw -l app=manusclaw

# Common causes:
# - Missing API key secrets
# - Invalid configuration
# - Resource limits too low
```

### Error: `ImagePullBackOff`

**Cause:** Cannot pull the ManusClaw Docker image.

```bash
# Check image pull secret
kubectl get secrets -n manusclaw

# Create a pull secret if using a private registry
kubectl create secret docker-registry manusclaw-pull \
  --docker-server=ghcr.io \
  --docker-username=your-username \
  --docker-password=your-token

# Reference in deployment
# imagePullSecrets:
#   - name: manusclaw-pull
```

### Error: `Pods not scaling with HPA`

**Cause:** Metrics server is not installed or custom metrics are not configured.

```bash
# Install metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Check HPA status
kubectl get hpa -n manusclaw

# Check metrics availability
kubectl top pods -n manusclaw
```

---

## Rate Limiting Issues

### Error: `Rate limit hit for provider X`

**Cause:** The LLM provider's API rate limit has been reached.

```bash
# Check which provider is rate-limited
manusclaw credentials status

# Solutions:
# 1. Wait for cooldown (typically 60 seconds)
# 2. Add more API keys to the credential pool
# 3. Switch to a different provider
# 4. Use a model failover profile to automatically switch
```

---

## Token Budget Exhausted

### Error: `Token budget exceeded`

**Cause:** The total token usage has exceeded the configured budget.

```bash
# Check current token usage
manusclaw context status

# Increase token budget
# In config.toml:
# [token_budget]
# max_total_tokens = 500000

# Enable auto-summarization
manusclaw config set token_budget.auto_summarize true

# Use a cheaper model for large tasks
manusclaw --provider groq --model llama-3.3-70b-versatile "Large analysis task"
```

---

## How to Reset / Clean Install

If nothing else works, a clean install often resolves persistent issues:

```bash
# 1. Uninstall
pip uninstall manusclaw -y

# 2. Remove config (BACK UP FIRST!)
cp -r ~/.manusclaw ~/.manusclaw-backup
rm -rf ~/.manusclaw

# 3. Remove virtual environment
rm -rf ~/manusclaw-env

# 4. Clear pip cache
pip cache purge

# 5. Reinstall
python3 -m venv ~/manusclaw-env
source ~/manusclaw-env/bin/activate
pip install "manusclaw[all]"

# 6. Configure from scratch
manusclaw --validate-config

# 7. Test
manusclaw "Hello, are you working?"
```
