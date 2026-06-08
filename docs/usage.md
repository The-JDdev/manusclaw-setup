# Usage Guide — ManusClaw v5.0.0

This guide covers everything you need to know about using ManusClaw v5.0.0 on a day-to-day basis. From launching your first session to advanced features like multi-agent orchestration, voice commands, SSH gateways, webhook management, cron scheduling, model failover, config profiles, and the skills system — every feature is explained with practical examples.

---

## Table of Contents

- [Entry Point Commands](#entry-point-commands)
  - [manusclaw — The Main Agent](#manusclaw--the-main-agent)
  - [manusclaw-server — HTTP API Server](#manusclaw-server--http-api-server)
  - [manusclaw-multi — Multi-Agent Orchestrator](#manusclaw-multi--multi-agent-orchestrator)
  - [manusclaw-cron — Scheduled Task Runner](#manusclaw-cron--scheduled-task-runner)
  - [manusclaw-sessions — Session Management CLI](#manusclaw-sessions--session-management-cli)
  - [manusclaw-channels — Channel Management CLI](#manusclaw-channels--channel-management-cli)
  - [manusclaw-webhook — Webhook Management CLI](#manusclaw-webhook--webhook-management-cli)
  - [manusclaw voice wake — Voice Wake Mode](#manusclaw-voice-wake--voice-wake-mode)
  - [manusclaw voice talk — Voice Talk Mode](#manusclaw-voice-talk--voice-talk-mode)
  - [manusclaw-ssh start — SSH Gateway](#manusclaw-ssh-start--ssh-gateway)
- [CLI Flags Reference](#cli-flags-reference)
- [Config Profiles](#config-profiles)
- [Model Failover](#model-failover)
- [Interactive Shell](#interactive-shell)
- [Single-Shot Mode](#single-shot-mode)
- [Slash Commands Reference](#slash-commands-reference)
- [Background Task Execution](#background-task-execution)
- [Task Queue Management](#task-queue-management)
- [Persistent Task System](#persistent-task-system)
- [Session Management](#session-management)
- [Memory System](#memory-system)
- [Skills System](#skills-system)
- [Tool Reference](#tool-reference)
- [Voice Commands](#voice-commands)
- [SSH Gateway Usage](#ssh-gateway-usage)
- [Webhook Management](#webhook-management)
- [Session Management CLI](#session-management-cli-1)
- [Cron Job Management](#cron-job-management)
- [Channel Management](#channel-management)
- [Server Endpoints Reference](#server-endpoints-reference)
- [Advanced Usage Patterns](#advanced-usage-patterns)

---

## Entry Point Commands

ManusClaw v5.0.0 provides **eleven entry points**, each serving a different purpose. Every command supports the global flags `--skin`, `--model`, `--profile`, `--no-color`, and `--version` (documented in [CLI Flags Reference](#cli-flags-reference)).

---

### manusclaw — The Main Agent

The primary way to interact with ManusClaw is through the `manusclaw` command, which launches an interactive shell where you can have a conversation with the AI agent.

```bash
# Launch the interactive shell (uses default config profile)
manusclaw

# Launch with a specific workspace
manusclaw --workspace /path/to/project

# Launch with a specific config file
manusclaw --config /path/to/config.toml

# Launch with a specific provider
manusclaw --provider anthropic --model claude-sonnet-4-20250514

# Launch in BUILD mode
manusclaw --mode BUILD

# Launch with a named config profile
manusclaw --profile production

# Launch with a specific UI skin
manusclaw --skin monokai

# Launch without color output (useful for logging or pipes)
manusclaw --no-color
```

On first launch, ManusClaw:
1. Creates the `~/.manusclaw/` configuration directory if it doesn't exist
2. Generates default `config.toml` and `.env` files
3. Initializes the workspace directory
4. Loads any existing MEMORY.md and USER.md files
5. Displays the current configuration summary

---

### manusclaw-server — HTTP API Server

Run ManusClaw as a persistent HTTP server, accessible via REST API and WebSocket connections. The server exposes chat, canvas, multi-agent, webhook, and health endpoints.

```bash
# Start the server on the default port (8000)
manusclaw-server

# Start with custom host and port
manusclaw-server --host 0.0.0.0 --port 9000

# Start with API key authentication
manusclaw-server --api-key your-secret-key

# Start with multiple workers
manusclaw-server --workers 4

# Start with a config profile
manusclaw-server --profile production

# Start with a specific model override
manusclaw-server --model claude-sonnet-4-20250514

# Start in development mode (auto-reload, verbose logging)
manusclaw-server --dev

# Start with CORS enabled for browser-based clients
manusclaw-server --cors "*"
```

The server supports both REST endpoints and persistent WebSocket connections for real-time streaming. See [Server Endpoints Reference](#server-endpoints-reference) for the full list of routes.

---

### manusclaw-multi — Multi-Agent Orchestrator

Run multiple agent instances in parallel for complex tasks. The orchestrator distributes work across agents using configurable strategies, manages inter-agent communication, and aggregates results.

```bash
# Start multi-agent mode with default configuration
manusclaw-multi

# Start with a specific number of agents
manusclaw-multi --agents 3

# Start with a specific task distribution strategy
manusclaw-multi --strategy round-robin

# Start with a shared workspace
manusclaw-multi --workspace /path/to/project

# Start with a config profile
manusclaw-multi --profile production

# Start with model failover enabled
manusclaw-multi --failover

# Start with an orchestration plan from a file
manusclaw-multi --plan /path/to/plan.yaml

# Start with agent-specific roles
manusclaw-multi --roles "researcher,coder,reviewer"
```

**Available strategies:**

| Strategy | Description |
|----------|-------------|
| `round-robin` | Tasks are distributed to agents in rotation |
| `specialist` | Tasks are routed to the agent best suited based on skill tags |
| `broadcast` | Every agent receives every task; best answer is selected |
| `pipeline` | Agents form a pipeline where output of one feeds into the next |

---

### manusclaw-cron — Scheduled Task Runner

Run tasks on a recurring schedule using cron expressions. The cron daemon runs in the background and executes ManusClaw tasks at specified intervals.

```bash
# Start the cron daemon
manusclaw-cron

# List all scheduled tasks
manusclaw-cron --list

# Add a new scheduled task
manusclaw-cron --add "0 9 * * 1" "Summarize the weekly meeting notes"

# Add a task with a named profile
manusclaw-cron --add "0 8 * * *" "Run daily security scan" --profile security

# Remove a scheduled task by ID
manusclaw-cron --remove task_001

# Remove a scheduled task by name
manusclaw-cron --remove --name "daily-security-scan"

# Pause all scheduled tasks
manusclaw-cron --pause

# Resume all scheduled tasks
manusclaw-cron --resume

# Show execution history for a task
manusclaw-cron --history task_001

# Export the current cron schedule
manusclaw-cron --export schedule.yaml

# Import a cron schedule from a file
manusclaw-cron --import schedule.yaml

# Validate cron expressions without scheduling
manusclaw-cron --validate "0 */2 * * *"
```

**Cron expression format:** `minute hour day-of-month month day-of-week`

| Expression | Meaning |
|------------|---------|
| `0 9 * * 1` | Every Monday at 9:00 AM |
| `*/30 * * * *` | Every 30 minutes |
| `0 8,20 * * *` | Every day at 8:00 AM and 8:00 PM |
| `0 0 1 * *` | First day of every month at midnight |
| `0 9 * * 1-5` | Every weekday at 9:00 AM |

See [Cron Job Management](#cron-job-management) for detailed usage including task logging, failure handling, and chained cron jobs.

---

### manusclaw-sessions — Session Management CLI

Manage ManusClaw sessions from the command line without entering the interactive shell. This is useful for scripting, automation, and CI/CD workflows.

```bash
# List all sessions
manusclaw-sessions list

# List sessions with details (messages, tokens, duration)
manusclaw-sessions list --verbose

# List sessions matching a pattern
manusclaw-sessions list --filter "api-refactor*"

# Show detailed info about a specific session
manusclaw-sessions info session_abc123

# Show the conversation history of a session
manusclaw-sessions history session_abc123

# Show the last N messages of a session
manusclaw-sessions history session_abc123 --limit 20

# Send a message to an active session
manusclaw-sessions send session_abc123 "What files did we change yesterday?"

# Send a message and get the response (non-interactive)
manusclaw-sessions send session_abc123 "Summarize our progress" --wait

# Spawn a new session from the CLI
manusclaw-sessions spawn --name "code-review-session" --workspace /path/to/project

# Spawn a session with a system prompt
manusclaw-sessions spawn --name "debug-session" --prompt "Focus on debugging React components"

# Export a session to a file
manusclaw-sessions export session_abc123 --output session_export.json

# Import a session from a file
manusclaw-sessions import session_export.json

# Delete a session
manusclaw-sessions delete session_abc123

# Delete all sessions older than 30 days
manusclaw-sessions prune --older-than 30d

# Show session statistics
manusclaw-sessions stats
```

See [Session Management CLI](#session-management-cli-1) for more detailed examples and automation patterns.

---

### manusclaw-channels — Channel Management CLI

Manage communication channels for ManusClaw. Channels are named message streams that can connect ManusClaw to external services like Slack, Discord, email, or custom integrations.

```bash
# List all configured channels
manusclaw-channels list

# Show details of a specific channel
manusclaw-channels info slack-primary

# Create a new channel
manusclaw-channels create \
  --name slack-primary \
  --type slack \
  --webhook https://hooks.slack.com/services/T00/B00/xxx \
  --events "task.complete,task.fail"

# Create a Discord channel
manusclaw-channels create \
  --name discord-alerts \
  --type discord \
  --webhook https://discord.com/api/webhooks/xxx/yyy \
  --events "error,cron.*"

# Update an existing channel
manusclaw-channels update slack-primary --events "task.*,error,cron.*"

# Enable a channel
manusclaw-channels enable slack-primary

# Disable a channel
manusclaw-channels disable slack-primary

# Delete a channel
manusclaw-channels delete slack-primary

# Test a channel by sending a test message
manusclaw-channels test slack-primary

# Show channel event logs
manusclaw-channels logs slack-primary --limit 50

# List available channel types
manusclaw-channels types
```

**Supported channel types:**

| Type | Description |
|------|-------------|
| `slack` | Slack incoming webhooks and bot integration |
| `discord` | Discord webhooks and bot integration |
| `email` | SMTP-based email notifications |
| `webhook` | Generic HTTP webhook endpoints |
| `pagerduty` | PagerDuty incident management |
| `telegram` | Telegram bot integration |
| `stdout` | Output to terminal (useful for debugging) |

See [Channel Management](#channel-management) for detailed configuration and automation patterns.

---

### manusclaw-webhook — Webhook Management CLI

Manage ManusClaw's webhook endpoints. Webhooks allow external systems to trigger ManusClaw tasks, receive notifications, and integrate ManusClaw into your CI/CD pipelines and automation workflows.

```bash
# List all registered webhooks
manusclaw-webhook list

# Show details of a specific webhook
manusclaw-webhook info wh_github_pr

# Register a new webhook endpoint
manusclaw-webhook create \
  --name wh_github_pr \
  --url /webhooks/github \
  --secret my-webhook-secret \
  --events "push,pull_request" \
  --action "Review the code changes and suggest improvements"

# Register a webhook with a config profile
manusclaw-webhook create \
  --name wh_deploy \
  --url /webhooks/deploy \
  --secret deploy-secret \
  --action "Run deployment checks" \
  --profile production

# Update a webhook's configuration
manusclaw-webhook update wh_github_pr --action "Run full code review pipeline"

# Enable a webhook
manusclaw-webhook enable wh_github_pr

# Disable a webhook
manusclaw-webhook disable wh_github_pr

# Delete a webhook
manusclaw-webhook delete wh_github_pr

# Rotate a webhook's secret
manusclaw-webhook rotate-secret wh_github_pr

# Show webhook delivery logs
manusclaw-webhook logs wh_github_pr --limit 20

# Replay a failed webhook delivery
manusclaw-webhook replay wh_github_pr --delivery-id del_12345

# Test a webhook with sample payload
manusclaw-webhook test wh_github_pr --payload '{"ref": "refs/heads/main"}'

# Show webhook statistics (deliveries, successes, failures)
manusclaw-webhook stats
```

See [Webhook Management](#webhook-management) for detailed configuration and CI/CD integration examples.

---

### manusclaw voice wake — Voice Wake Mode

ManusClaw v5.0.0 introduces voice interaction support. The `voice wake` command starts a background listener that activates ManusClaw when it detects a wake word, similar to voice assistants.

```bash
# Start voice wake mode with default wake word ("Hey ManusClaw")
manusclaw voice wake

# Start with a custom wake word
manusclaw voice wake --wake-word "Hey Claw"

# Start with a specific audio input device
manusclaw voice wake --device 2

# Start with a specific language model for speech recognition
manusclaw voice wake --language en-US

# Start with sensitivity tuning (0.0–1.0)
manusclaw voice wake --sensitivity 0.7

# Start with a config profile
manusclaw voice wake --profile production

# Start in debug mode to see audio processing details
manusclaw voice wake --debug
```

When the wake word is detected, ManusClaw will:
1. Play an audible acknowledgment tone
2. Begin recording your voice command
3. Transcribe the audio to text using the configured ASR provider
4. Execute the transcribed command as if typed into the interactive shell
5. Optionally speak the response back to you (requires TTS configuration)

**Note:** Voice wake mode requires an active `manusclaw-server` instance running, or you can pair it directly with a local `manusclaw` session.

---

### manusclaw voice talk — Voice Talk Mode

Voice talk mode provides a continuous voice conversation with ManusClaw. Unlike voice wake mode (which is passive and waits for a wake word), voice talk mode is an always-on, back-and-forth voice conversation.

```bash
# Start voice talk mode
manusclaw voice talk

# Start with a specific model
manusclaw voice talk --model claude-sonnet-4-20250514

# Start with a config profile
manusclaw voice talk --profile production

# Start with push-to-talk mode (hold Space to talk)
manusclaw voice talk --push-to-talk

# Start with automatic turn detection (voice activity detection)
manusclaw voice talk --auto-detect

# Start with a specific TTS voice
manusclaw voice talk --tts-voice "en-US-Neural2-D"

# Start with verbose transcription output
manusclaw voice talk --verbose-transcription

# Start with noise cancellation enabled
manusclaw voice talk --noise-cancel
```

**Voice talk controls during conversation:**

| Control | Action |
|---------|--------|
| Hold `Space` | Talk (push-to-talk mode) |
| `Ctrl+C` | Stop current generation |
| `Ctrl+D` | Exit voice talk mode |
| `Ctrl+M` | Mute microphone |
| `Ctrl+S` | Toggle speech output on/off |
| `Ctrl+L` | Clear conversation context |

See [Voice Commands](#voice-commands) for the full list of voice commands and configuration options.

---

### manusclaw-ssh start — SSH Gateway

The SSH gateway allows you to access ManusClaw remotely over SSH. This is useful for connecting to ManusClaw from another machine, or for integrating ManusClaw into SSH-based workflows.

```bash
# Start the SSH gateway on the default port (2222)
manusclaw-ssh start

# Start on a custom port
manusclaw-ssh start --port 3000

# Start with password authentication
manusclaw-ssh start --auth password

# Start with public key authentication only
manusclaw-ssh start --auth pubkey

# Start with a specific host key
manusclaw-ssh start --host-key /path/to/ssh_host_key

# Start with a config profile
manusclaw-ssh start --profile production

# Start with a whitelist of allowed users
manusclaw-ssh start --allowed-users "alice,bob,charlie"

# Start with connection logging enabled
manusclaw-ssh start --log-connections

# Start in background (daemon mode)
manusclaw-ssh start --daemon

# Start with max concurrent sessions
manusclaw-ssh start --max-sessions 10
```

Once the SSH gateway is running, connect to it from any SSH client:

```bash
# Connect to the ManusClaw SSH gateway
ssh -p 2222 localhost

# Connect from a remote machine
ssh -p 2222 user@your-server-ip
```

After connecting, you get a full ManusClaw interactive shell over SSH, identical to running `manusclaw` locally.

See [SSH Gateway Usage](#ssh-gateway-usage) for detailed configuration, authentication, and security information.

---

## CLI Flags Reference

ManusClaw v5.0.0 supports a unified set of global CLI flags that can be used with any entry point command.

### Global Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--skin <name>` | Apply a named UI skin/theme for terminal output | `manusclaw --skin monokai` |
| `--model <model>` | Override the default LLM model for this session | `manusclaw --model claude-sonnet-4-20250514` |
| `--profile <name>` | Load a specific configuration profile | `manusclaw --profile production` |
| `--no-color` | Disable all color and formatting in output | `manusclaw --no-color` |
| `--version` | Print version information and exit | `manusclaw --version` |
| `--help` | Print help information and exit | `manusclaw --help` |

### Common Flags (available on most entry points)

| Flag | Description | Entry Points |
|------|-------------|-------------|
| `--workspace <path>` | Set the working directory | `manusclaw`, `manusclaw-server`, `manusclaw-multi` |
| `--config <path>` | Path to a custom config.toml file | All entry points |
| `--provider <name>` | Override the LLM provider | `manusclaw`, `manusclaw-server`, `manusclaw-multi` |
| `--mode <mode>` | Set the permission mode (PLAN or BUILD) | `manusclaw`, `manusclaw-server` |
| `--verbose` | Enable verbose output / debug logging | All entry points |
| `--quiet` | Suppress all non-essential output | All entry points |

### Server-Specific Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--host <addr>` | Bind address for the HTTP server | `manusclaw-server --host 0.0.0.0` |
| `--port <num>` | Port for the HTTP server | `manusclaw-server --port 9000` |
| `--api-key <key>` | API key for authentication | `manusclaw-server --api-key mykey` |
| `--workers <n>` | Number of worker processes | `manusclaw-server --workers 4` |
| `--cors <origin>` | CORS allowed origins | `manusclaw-server --cors "*"` |
| `--dev` | Development mode (auto-reload) | `manusclaw-server --dev` |

### Output

```bash
$ manusclaw --version
ManusClaw v5.0.0
Python 3.12.4
Installation: /home/user/.local/lib/python3.12/site-packages/manusclaw
```

---

## Config Profiles

ManusClaw v5.0.0 introduces configuration profiles, allowing you to maintain multiple named configurations and switch between them instantly. Profiles are defined in `config.toml` under the `[profiles]` section, or by using the `MANUSCLAW_PROFILE` environment variable.

### Defining Profiles in config.toml

```toml
[profiles.development]
provider = "openai"
model = "gpt-4o"
mode = "PLAN"
workspace = "/home/user/dev/myproject"
api_key = "${OPENAI_DEV_KEY}"

[profiles.production]
provider = "anthropic"
model = "claude-sonnet-4-20250514"
mode = "BUILD"
workspace = "/opt/myproject"
api_key = "${ANTHROPIC_PROD_KEY}"
workers = 8
log_level = "warning"

[profiles.local]
provider = "ollama"
model = "llama3"
mode = "PLAN"
workspace = "/home/user/projects/test"
base_url = "http://localhost:11434"

[profiles.security]
provider = "anthropic"
model = "claude-sonnet-4-20250514"
mode = "PLAN"
tools = ["file_read", "file_search", "web_search", "web_fetch"]
disabled_tools = ["shell_exec", "file_write", "file_delete"]
```

### Using Profiles

```bash
# Select a profile with --profile flag
manusclaw --profile production

# Select a profile with environment variable
MANUSCLAW_PROFILE=production manusclaw

# Use a profile with the server
manusclaw-server --profile production --workers 8

# Use a profile with multi-agent
manusclaw-multi --profile production --agents 5

# Use a profile with cron jobs
manusclaw-cron --profile security

# Use a profile with SSH gateway
manusclaw-ssh start --profile production

# Use a profile with voice
manusclaw voice talk --profile local
```

### Profile Precedence

When multiple configuration sources are active, the precedence order is (highest to lowest):

1. **CLI flags** (e.g., `--model`, `--provider`) — always win
2. **Environment variables** (e.g., `MANUSCLAW_PROFILE`, `MANUSCLAW_MODEL`)
3. **Active profile** (from `--profile` or `MANUSCLAW_PROFILE`)
4. **Global defaults** (from the `[default]` section of config.toml)

### Profile Inheritance

Profiles can inherit from other profiles using the `inherits` key:

```toml
[profiles.base]
provider = "anthropic"
model = "claude-sonnet-4-20250514"
mode = "PLAN"

[profiles.production]
inherits = "base"
mode = "BUILD"
workers = 8
api_key = "${ANTHROPIC_PROD_KEY}"

[profiles.staging]
inherits = "production"
mode = "PLAN"
workspace = "/opt/staging"
```

In this example, `staging` inherits all settings from `production`, which in turn inherits from `base`. The `staging` profile overrides `mode` and `workspace`.

---

## Model Failover

ManusClaw v5.0.0 supports automatic model failover, ensuring that your workflows continue running even when a primary LLM provider is unavailable, rate-limited, or returning errors. Failover chains are configured in `config.toml`.

### Configuring Failover Chains

```toml
[failover]
enabled = true
# List models in priority order. If the first fails, ManusClaw tries the next.
chain = [
  "anthropic/claude-sonnet-4-20250514",
  "openai/gpt-4o",
  "anthropic/claude-haiku-3-5-20241022",
  "ollama/llama3"
]
# Number of retries before failing over to the next model
max_retries = 3
# Delay between retries in seconds
retry_delay = 2
# Conditions that trigger failover
failover_on = ["rate_limit", "timeout", "server_error", "auth_error"]
# Whether to fall back after N consecutive failures
consecutive_failures = 2
```

### Using Failover from the CLI

```bash
# Enable failover with --failover flag
manusclaw --failover

# Enable failover with a specific chain
manusclaw --failover --failover-chain "anthropic/claude-sonnet-4-20250514,openai/gpt-4o,ollama/llama3"

# Enable failover on the server
manusclaw-server --failover

# Enable failover on multi-agent
manusclaw-multi --failover --agents 4

# Use a profile that has failover configured
manusclaw --profile production  # production profile includes failover config
```

### Failover Behavior

When failover is active, ManusClaw monitors every LLM API call:

1. **Primary model attempt** — ManusClaw sends the request to the first model in the chain.
2. **Retry on transient errors** — If the request fails with a retryable error (rate limit, timeout, server error), ManusClaw retries up to `max_retries` times with `retry_delay` between attempts.
3. **Failover to next model** — If all retries are exhausted, ManusClaw moves to the next model in the chain and repeats the process.
4. **Exhaustion** — If all models in the chain fail, ManusClaw returns an error and logs the failure details.

During a failover event, you'll see a notification in the shell:

```
⚠️  Model failover: anthropic/claude-sonnet-4-20250514 → openai/gpt-4o
   Reason: rate_limit (429 Too Many Requests)
```

### Failover-Aware Slash Commands

The `/model` command in v5.0.0 is failover-aware:

```
# Show the current active model and the full failover chain
> /model
Active model: anthropic/claude-sonnet-4-20250514
Failover chain: anthropic/claude-sonnet-4-20250514 → openai/gpt-4o → ollama/llama3

# Switch to a specific model (overrides failover)
> /model openai/gpt-4o
Switched to: openai/gpt-4o (failover disabled for this session)

# Re-enable failover with the configured chain
> /model --failover
Failover re-enabled. Chain: anthropic/claude-sonnet-4-20250514 → openai/gpt-4o → ollama/llama3

# Set a new failover chain at runtime
> /model --chain "anthropic/claude-sonnet-4-20250514,anthropic/claude-haiku-3-5-20241022"
Failover chain updated: anthropic/claude-sonnet-4-20250514 → anthropic/claude-haiku-3-5-20241022
```

---

## Interactive Shell

When you launch `manusclaw` without arguments, you enter the interactive shell. This is a rich, terminal-based interface with auto-completion, syntax highlighting, and multi-line input support.

### First launch experience

```bash
$ manusclaw

╭──────────────────────────────────────────────────╮
│  ManusClaw v5.0.0                                 │
│  Provider: openai / Model: gpt-4o                 │
│  Profile: default / Skin: default                  │
│  Workspace: /home/user/workspace                   │
│  Mode: PLAN                                       │
│  Failover: disabled                               │
╰──────────────────────────────────────────────────╯

> Hello! How can I help you today?
```

### Input modes

The interactive shell supports two input modes:

**Single-line mode (default):** Type your message and press Enter to send it.

```
> What files are in the workspace?
```

**Multi-line mode:** Press `Ctrl+P` (or `Esc` then `Enter` on some terminals) to toggle multi-line mode. This is useful for pasting code or writing long prompts:

```
> ╭─────────────────────────────────╮
  │ Multi-line mode (Ctrl+P to send)│
  │                                 │
  │ Write a Python function that:   │
  │ 1. Reads a CSV file             │
  │ 2. Filters rows by date         │
  │ 3. Outputs a summary            │
  │                                 │
  ╰─────────────────────────────────╯
```

### Shell keyboard shortcuts

| Shortcut | Action |
|----------|--------|
| `Enter` | Send message (single-line mode) |
| `Ctrl+P` | Toggle multi-line mode / Send in multi-line |
| `Ctrl+C` | Cancel current input or interrupt agent |
| `Ctrl+D` | Exit ManusClaw |
| `Tab` | Auto-complete slash commands and file paths |
| `Up/Down` | Navigate command history |
| `Ctrl+R` | Search command history |
| `Ctrl+L` | Clear screen |
| `Ctrl+A` | Move cursor to start of line |
| `Ctrl+E` | Move cursor to end of line |

### Output formatting

ManusClaw uses Rich for beautiful, formatted output. The agent's responses include:

- **Syntax-highlighted code blocks** with language detection
- **Tables** for structured data
- **File trees** for directory listings
- **Progress bars** for long-running operations
- **Diff views** for code changes
- **Markdown rendering** for formatted text

---

## Single-Shot Mode

Single-shot mode lets you run a quick query without entering the interactive shell. This is useful for scripting, CI/CD pipelines, or quick one-off questions.

### Basic usage

```bash
# Ask a single question
manusclaw "What is the capital of France?"

# Run a coding task
manusclaw "Create a Python Flask app with a health check endpoint"

# Analyze a file
manusclaw "Explain what this code does: main.py"

# With a specific provider
manusclaw --provider anthropic "Write a haiku about debugging"

# With a config profile
manusclaw --profile production "Run the full test suite"
```

### Piping input

You can pipe text into ManusClaw for analysis:

```bash
# Analyze log files
cat /var/log/app.log | manusclaw "Summarize the errors in this log file"

# Review code
git diff | manusclaw "Review this code change and suggest improvements"

# Process JSON data
curl -s https://api.example.com/data | manusclaw "Extract the top 5 results"
```

### Using in scripts

Single-shot mode integrates naturally into shell scripts:

```bash
#!/bin/bash
# Auto-generate commit messages
DIFF=$(git diff --cached)
MESSAGE=$(manusclaw "Write a concise git commit message for these changes: $DIFF")
git commit -m "$MESSAGE"
```

```bash
#!/bin/bash
# Daily code review with a specific profile
manusclaw --profile security \
  "Review all Python files in the workspace for potential bugs and security issues" \
  > review-$(date +%Y%m%d).md
```

### Exit codes

| Code | Meaning |
|------|---------|
| 0 | Success — the agent completed the task |
| 1 | General error — something went wrong |
| 2 | Configuration error — check your config.toml or .env |
| 3 | API error — the LLM provider returned an error |
| 4 | Token budget exhausted — the conversation exceeded the token limit |
| 130 | Interrupted — the user pressed Ctrl+C |

---

## Slash Commands Reference

Slash commands are special instructions that start with `/` and are typed directly into the interactive shell. They provide quick access to common functions without leaving the conversation.

### General Commands

#### `/help` — Show help information

Displays a list of all available slash commands with brief descriptions.

```
> /help
```

#### `/exit` — Exit ManusClaw

Gracefully exits ManusClaw, saving any unsaved memory and session data.

```
> /exit
Saving memory... Done.
Goodbye!
```

### Model and Configuration Commands

#### `/model` — View or switch LLM models

The `/model` command in v5.0.0 is the primary way to manage your active model and failover chain.

```
# Show the current active model and failover chain
> /model
Active: anthropic/claude-sonnet-4-20250514
Failover chain: anthropic/claude-sonnet-4-20250514 → openai/gpt-4o → ollama/llama3

# Switch to a different model
> /model openai/gpt-4o
Switched to: openai/gpt-4o (failover disabled)

# Enable failover
> /model --failover
Failover enabled. Chain restored.

# Set a new failover chain
> /model --chain "anthropic/claude-sonnet-4-20250514,openai/gpt-4o,anthropic/claude-haiku-3-5-20241022"
Chain updated.
```

#### `/config` — View or modify configuration

```
# Show current configuration
> /config

# Show a specific section
> /config llm

# Change a setting
> /config llm.model gpt-4o-mini

# Switch providers
> /config llm.provider anthropic
> /config llm.model claude-sonnet-4-20250514
```

#### `/mode` — Switch permission mode

```
# Switch to PLAN mode (safer, requires confirmation)
> /mode PLAN
Permission mode set to PLAN. All actions will require confirmation.

# Switch to BUILD mode (autonomous)
> /mode BUILD
Permission mode set to BUILD. Actions will execute automatically.
⚠️  Be careful: the agent can now execute actions without confirmation.
```

### Skills and Tools Commands

#### `/skills` — Manage skills

```
# List available skills from the registry
> /skills available

╭───────────────────────────────────────────────────────────╮
│ Skill Name         │ Description                         │
├────────────────────┼─────────────────────────────────────┤
│ web-dev            │ Full-stack web development          │
│ data-analysis      │ Data analysis and visualization     │
│ devops             │ DevOps and infrastructure           │
│ security           │ Security auditing and testing       │
│ documentation      │ Technical writing and docs          │
╰───────────────────────────────────────────────────────────╯

# List installed skills
> /skills list

# Install a skill
> /skills install web-dev
Skill 'web-dev' installed successfully.

# Install from a URL
> /skills install https://github.com/user/manusclaw-skill-custom

# Enable a skill
> /skills enable web-dev

# Disable a skill
> /skills disable web-dev

# Update a skill
> /skills update web-dev

# Update all skills
> /skills update --all

# Uninstall a skill
> /skills uninstall web-dev
```

#### `/tools` — List available tools

```
> /tools

╭──────────────────────────────────────────────────╮
│ Tool Name     │ Description                      │
├───────────────┼──────────────────────────────────┤
│ file_read     │ Read file contents               │
│ file_write    │ Write or modify files            │
│ file_delete   │ Delete files                     │
│ shell_exec    │ Execute shell commands           │
│ web_search    │ Search the web                   │
│ web_browse    │ Browse web pages                 │
│ code_execute  │ Execute code in sandbox          │
│ memory_write  │ Write to memory file             │
│ skill_install │ Install new skills               │
│ canvas_draw   │ Draw on the shared canvas         │
│ session_spawn │ Spawn a new session               │
╰──────────────────────────────────────────────────╯
```

### Task Management Commands

#### `/bg` — Run a task in the background

The `/bg` command allows you to start a long-running task and continue working in the foreground while the background task executes independently.

```
# Run a background task
> /bg Analyze all Python files in the workspace and create a dependency graph

Background task started (ID: task_abc123)
You can continue working while this task runs.
Use /tasks to check progress.
```

Background tasks are persistent — they continue running even if you start a new conversation or switch topics. You can have multiple background tasks running simultaneously.

**When to use /bg:**
- Long code analysis tasks that don't need your immediate attention
- Batch file operations (renaming, reformatting, refactoring)
- Generating documentation for an entire codebase
- Running test suites and collecting results
- Any task that takes more than a few seconds and doesn't require interactive input

#### `/tasks` — List and manage tasks

```
# List all tasks
> /tasks

╭─────────────────────────────────────────────────────────╮
│ Task ID       │ Status    │ Description                  │
├───────────────┼───────────┼──────────────────────────────┤
│ task_abc123   │ RUNNING   │ Analyze Python dependencies  │
│ task_def456   │ COMPLETED │ Generate API documentation   │
│ task_ghi789   │ FAILED    │ Deploy to staging server     │
╰─────────────────────────────────────────────────────────╯

# View details of a specific task
> /tasks task_abc123

# View output of a completed task
> /tasks task_def456 --output

# Cancel a running task
> /tasks task_abc123 --cancel

# Retry a failed task
> /tasks task_ghi789 --retry
```

### Memory and Context Commands

#### `/memory` — View or edit agent memory

```
# Show current memory contents
> /memory

# Show memory with line numbers
> /memory --lines

# Clear all memory
> /memory --clear

# Add a memory entry manually
> /memory add "This project uses Django 4.2 with PostgreSQL"

# Search memory
> /memory search "database"
```

#### `/compress` — Compress conversation context

The `/compress` command summarizes the current conversation context, reducing token usage while preserving key information. This is especially useful in long sessions where the context window is filling up.

```
> /compress

Context compressed.
  Before: 45,230 tokens (22.6% of budget)
  After:  12,450 tokens (6.2% of budget)
  Compression ratio: 72.5%

Key points preserved:
  - Project uses Django 4.2 with PostgreSQL
  - We refactored the auth module to use JWT
  - 3 remaining tasks in the queue
  - User prefers concise explanations
```

When to use `/compress`:
- When the token usage indicator shows > 50% of the budget
- Before starting a new topic within the same session
- When the agent starts losing focus due to long context
- Periodically during extended sessions

#### `/user` — View or edit user profile

```
# Show user profile
> /user

# Set a preference
> /user set preferred_language "Python"
> /user set coding_style "PEP 8 with 120 character line limit"
> /user set experience_level "senior"
```

### Session and Conversation Commands

#### `/new` — Start a new conversation

Start a fresh conversation in the same ManusClaw instance without exiting. Memory and user profile are preserved.

```
> /new
Starting new conversation. Memory and user profile preserved.
Session: session_xyz789
```

#### `/resume` — Resume a previous session

Resume a previously saved session, restoring the full conversation history and context.

```
# Resume the most recent session
> /resume
Resuming session: api-refactor-session (45 messages)

# Resume a specific session
> /resume api-refactor-session
Resuming session: api-refactor-session

# List available sessions to resume
> /resume --list
```

#### `/branch` — Branch the conversation

Create a branch from the current conversation point, allowing you to explore different directions without losing the original context.

```
# Create a branch from the current point
> /branch
Branch created: branch_001
You are now on branch_001. The original conversation is preserved.

# Create a named branch
> /branch "explore-react-approach"
Branch created: explore-react-approach

# Switch back to the main conversation
> /branch --switch main

# List all branches
> /branch --list

╭──────────────────────────────────────────────────────╮
│ Branch                │ Messages │ Created           │
├───────────────────────┼──────────┼───────────────────┤
│ main                  │ 23       │ 2025-01-15 10:00  │
│ branch_001            │ 5        │ 2025-01-15 10:30  │
│ explore-react-approach│ 12       │ 2025-01-15 11:00  │
╰──────────────────────────────────────────────────────╯

# Merge a branch back into main
> /branch --merge explore-react-approach
Branch 'explore-react-approach' merged into main. 12 messages added.
```

### Session Slash Commands (v5.0.0)

These commands mirror the functionality of `manusclaw-sessions` CLI but are available directly in the interactive shell.

#### `/sessions list` — List sessions

```
# List all sessions
> /sessions list

╭──────────────────────────────────────────────────────────────────╮
│ Session ID            │ Name               │ Date       │ Status  │
├───────────────────────┼────────────────────┼────────────┼─────────┤
│ session_abc123        │ api-refactor       │ 2025-01-15 │ active  │
│ session_def456        │ code-review        │ 2025-01-15 │ idle    │
│ session_ghi789        │ debug-session      │ 2025-01-14 │ saved   │
╰──────────────────────────────────────────────────────────────────╯

# List with verbose details
> /sessions list --verbose
```

#### `/sessions history` — View session history

```
# View history of a specific session
> /sessions history session_abc123

# View the last N messages
> /sessions history session_abc123 --limit 10
```

#### `/sessions send` — Send a message to a session

```
# Send a message to another session
> /sessions send session_abc123 "What files did we change?"

# Send and wait for a response
> /sessions send session_abc123 "Summarize our progress" --wait
```

#### `/sessions spawn` — Spawn a new session

```
# Spawn a new session
> /sessions spawn --name "investigation" --workspace /path/to/project
Session spawned: session_jkl012 (name: investigation)

# Spawn a session with an initial prompt
> /sessions spawn --name "research" --prompt "Research the best authentication libraries for Python"
```

### File and Workspace Commands

#### `/workspace` — Workspace information

```
# Show current workspace info
> /workspace

# Change workspace
> /workspace /path/to/other/project

# Show workspace file tree
> /workspace --tree
```

#### `/files` — List workspace files

```
# List files in the workspace
> /files

# List files with details
> /files --long

# List only Python files
> /files --type py

# Search for files
> /files --search "config"
```

### Debug Commands

#### `/debug` — Toggle debug mode

```
> /debug on
Debug mode enabled. Detailed logging will be shown.

> /debug off
Debug mode disabled.
```

#### `/token-usage` — Show token usage statistics

```
> /token-usage

╭───────────────────────────────────────────────────╮
│ Session Token Usage                               │
├───────────────────────────────────────────────────┤
│ Input tokens:     45,230                          │
│ Output tokens:    12,450                          │
│ Total tokens:     57,680                          │
│ Budget used:      28.8% of 200,000               │
│ Estimated cost:   $0.43                           │
│ Failover events:  0                               │
╰───────────────────────────────────────────────────╯
```

---

## Background Task Execution

The background task system is one of ManusClaw's standout features. It allows you to offload long-running tasks to background workers while you continue working interactively.

### How background tasks work

When you use the `/bg` command, ManusClaw:

1. **Creates a new task** with a unique ID
2. **Spawns a background worker** that executes the task independently
3. **Returns control to you** immediately, so you can keep chatting
4. **Notifies you** when the task completes, fails, or needs input

The background worker has its own conversation context, separate from your main session. It can read and write files, search the web, and use all of ManusClaw's tools — but it won't interfere with what you're doing in the foreground.

### Examples

#### Example 1: Code review in the background

```
> /bg Review all Python files in the workspace for potential bugs, security issues, and code style violations. Create a report in workspace/review-report.md

Background task started (ID: task_code_review)
```

While the code review runs, you can continue asking questions or working on other tasks. When it finishes, you'll see a notification:

```
[Background] task_code_review completed. Output: workspace/review-report.md
```

#### Example 2: Running tests

```
> /bg Run the entire test suite, collect the results, and summarize which tests passed, failed, or were skipped

Background task started (ID: task_test_run)
```

#### Example 3: Batch file operations

```
> /bg Convert all JPEG images in the assets/ directory to WebP format with 80% quality

Background task started (ID: task_img_convert)
```

### Monitoring background tasks

```
# Check the status of all tasks
> /tasks

# Get detailed status of a specific task
> /tasks task_code_review

# View the output of a completed task
> /tasks task_code_review --output

# View real-time logs of a running task
> /tasks task_code_review --follow
```

### Cancelling background tasks

```
> /tasks task_code_review --cancel
Task task_code_review cancelled.
```

### Limits

- **Maximum concurrent background tasks:** 5 (configurable in config.toml)
- **Task timeout:** 30 minutes by default (configurable)
- **Memory:** Background tasks share the same memory system as the main session, so they can read and update MEMORY.md

---

## Task Queue Management

The task queue allows you to line up multiple tasks that will execute sequentially. This is different from background tasks, which run in parallel. The queue is useful when tasks need to be executed in order or when you want to limit resource consumption.

### Adding tasks to the queue

```
> /queue add "Generate unit tests for src/auth.py"
Task added to queue (position: 1)

> /queue add "Generate unit tests for src/api.py"
Task added to queue (position: 2)

> /queue add "Run all tests and report results"
Task added to queue (position: 3)
```

### Managing the queue

```
# View the queue
> /queue list

╭──────────────────────────────────────────────────────╮
│ Pos │ Task Description                        │ Status │
├─────┼─────────────────────────────────────────┼────────┤
│ 1   │ Generate unit tests for src/auth.py     │ ACTIVE │
│ 2   │ Generate unit tests for src/api.py      │ WAITING│
│ 3   │ Run all tests and report results        │ WAITING│
╰──────────────────────────────────────────────────────╯

# Remove a task from the queue
> /queue remove 2

# Reorder tasks
> /queue move 3 1

# Clear the entire queue
> /queue clear

# Pause the queue (finish current task, don't start next)
> /queue pause

# Resume the queue
> /queue resume
```

### Queue vs. Background Tasks

| Feature | Background Tasks (/bg) | Task Queue (/queue) |
|---------|----------------------|---------------------|
| **Execution** | Parallel | Sequential |
| **Use case** | Independent tasks | Ordered/dependent tasks |
| **Concurrency** | Up to 5 at once | 1 at a time |
| **Resource usage** | Higher (multiple workers) | Lower (single worker) |
| **Task dependency** | None | Previous task completes first |

---

## Persistent Task System

ManusClaw's persistent task system ensures that tasks survive across sessions. If you close your terminal or if ManusClaw crashes, any running or queued tasks can be recovered when you restart.

### How persistence works

Tasks and their state are saved to disk in the `~/.manusclaw/tasks/` directory. Each task is stored as a JSON file containing:

- The task ID and description
- The conversation context (messages exchanged so far)
- The current status (running, completed, failed)
- The output and any generated files
- Timestamps (created, started, completed)
- The config profile and model used

### Recovering tasks after a restart

```bash
# When you restart ManusClaw, it automatically detects unfinished tasks
$ manusclaw

⚠️  2 unfinished tasks detected:
  - task_code_review (status: interrupted)
  - task_docs (status: interrupted)

> /tasks

# Resume an interrupted task
> /tasks task_code_review --resume

# Or discard it
> /tasks task_code_review --discard
```

### Task persistence configuration

```toml
[tasks]
persist = true                      # Enable task persistence
storage_dir = "~/.manusclaw/tasks"  # Directory for task data
auto_resume = true                  # Auto-resume interrupted tasks on startup
max_retention = "30 days"           # How long to keep completed task data
```

---

## Session Management

Sessions allow you to save and restore entire conversation states, including the message history, configuration, and context. In v5.0.0, sessions are more powerful than ever with branching, spawning, and inter-session messaging.

### Saving a session

```
> /save
Session saved: session_20250115_143022

# Save with a descriptive name
> /save --name "api-refactor-session"
Session saved: api-refactor-session
```

### Loading a session

```
# List all saved sessions
> /load --list

╭──────────────────────────────────────────────────────────────────╮
│ Session Name           │ Date                │ Messages │ Tokens │
├────────────────────────┼─────────────────────┼──────────┼────────┤
│ api-refactor-session   │ 2025-01-15 14:30   │ 45       │ 12,340 │
│ session_20250115_0915  │ 2025-01-15 09:15   │ 23       │ 5,678  │
│ session_20250114_1630  │ 2025-01-14 16:30   │ 67       │ 28,901 │
╰──────────────────────────────────────────────────────────────────╯

# Load a specific session
> /load api-refactor-session
Session loaded. 45 messages restored.

# Load the most recent session
> /load --recent
```

### Auto-save

ManusClaw can automatically save sessions at regular intervals:

```toml
[sessions]
auto_save = true           # Enable auto-save
save_interval = 300        # Save every 5 minutes
max_sessions = 50          # Keep the 50 most recent sessions
storage_dir = "~/.manusclaw/sessions"
```

---

## Memory System

The memory system is what makes ManusClaw truly powerful for long-term projects. It allows the agent to remember information across sessions, so you don't have to repeat context every time you start a new conversation.

### How memory works

ManusClaw uses two memory files:

#### MEMORY.md — Agent memory

This file stores information the agent learns about your project during conversations. It's automatically updated as the agent works.

**Example MEMORY.md:**

```markdown
# Project Memory

## Project Overview
- This is a Django 4.2 web application for e-commerce
- Uses PostgreSQL 15 as the primary database
- Frontend is React with TypeScript

## Architecture
- Backend: Django REST Framework
- API versioning: URL-based (v1, v2)
- Authentication: JWT with refresh tokens
- File storage: AWS S3

## Key Decisions
- 2025-01-15: Decided to use Celery for background tasks instead of Django-Q
- 2025-01-12: Switched from pytest to unittest for simpler test setup

## Known Issues
- The payment webhook handler has a race condition (issue #234)
- Search API is slow for queries with many filters
```

#### USER.md — User profile

This file stores information about you — your preferences, coding style, and any personal context.

**Example USER.md:**

```markdown
# User Profile

## Preferences
- Preferred language: Python
- Framework: Django for web, FastAPI for APIs
- Coding style: PEP 8 with 120 character line limit
- Testing: pytest with factory_boy
- Version control: Git with conventional commits

## Communication
- I prefer concise explanations with code examples
- Skip basic explanations — I'm a senior developer
- Use type hints in all Python code

## Current Focus
- Migrating the monolith to microservices
- Improving test coverage above 80%
```

### Managing memory manually

```
# View current memory
> /memory

# Add a memory entry
> /memory add "The deployment pipeline uses GitHub Actions with staging and production environments"

# Search memory
> /memory search "deployment"

# Clear all memory (use with caution!)
> /memory --clear
Are you sure? This will delete all stored memory. (y/n): y
Memory cleared.
```

### Memory best practices

1. **Let the agent manage memory automatically.** The agent is good at deciding what's worth remembering.
2. **Review MEMORY.md periodically.** Check it every few sessions to make sure the information is still accurate.
3. **Keep USER.md up to date.** The agent reads it at the start of every session.
4. **Don't put secrets in memory files.** MEMORY.md and USER.md are plain text files.
5. **Commit memory files to git.** Consider committing them to your project's git repository.

---

## Skills System

Skills are modular extensions that add new capabilities to ManusClaw. They can be installed, enabled, and disabled without modifying the core framework.

### What are skills?

A skill is a collection of:
- **Prompt templates** — Specialized instructions for the agent
- **Tool definitions** — New tools the agent can use
- **Configuration** — Skill-specific settings

### Installing skills

```
# List available skills
> /skills available

╭───────────────────────────────────────────────────────────╮
│ Skill Name         │ Description                         │
├────────────────────┼─────────────────────────────────────┤
│ web-dev            │ Full-stack web development          │
│ data-analysis      │ Data analysis and visualization     │
│ devops             │ DevOps and infrastructure           │
│ security           │ Security auditing and testing       │
│ documentation      │ Technical writing and docs          │
╰───────────────────────────────────────────────────────────╯

# Install a skill
> /skills install web-dev
Skill 'web-dev' installed successfully.

# Install from a URL
> /skills install https://github.com/user/manusclaw-skill-custom
```

### Managing skills

```
# List installed skills
> /skills list

# Enable a skill
> /skills enable web-dev

# Disable a skill
> /skills disable web-dev

# Update a skill
> /skills update web-dev

# Update all skills
> /skills update --all

# Uninstall a skill
> /skills uninstall web-dev
```

### Creating custom skills

You can create your own skills by adding files to the `~/.manusclaw/skills/` directory:

```
~/.manusclaw/skills/
└── my-custom-skill/
    ├── manifest.toml        # Skill metadata and configuration
    ├── prompts/             # Prompt templates
    │   └── main.md
    ├── tools/               # Custom tool implementations
    │   └── my_tool.py
    └── config.toml          # Skill-specific configuration
```

The `manifest.toml` file describes the skill:

```toml
[skill]
name = "my-custom-skill"
version = "1.0.0"
description = "A custom skill for specialized tasks"
author = "Your Name"

[tools]
my_tool = { module = "tools.my_tool", description = "Does something custom" }
```

---

## Tool Reference

ManusClaw v5.0.0 provides a comprehensive set of built-in tools that the agent uses to accomplish tasks.

### File Operations

| Tool | Description | Permission Required |
|------|-------------|-------------------|
| `file_read` | Read the contents of a file | Read |
| `file_write` | Create or modify a file | Write |
| `file_delete` | Delete a file | Write |
| `file_search` | Search for files by name or content | Read |
| `file_tree` | Display directory structure | Read |
| `file_diff` | Show changes between file versions | Read |

### Shell and Code Execution

| Tool | Description | Permission Required |
|------|-------------|-------------------|
| `shell_exec` | Execute a shell command | Execute |
| `code_execute` | Execute code in a sandboxed environment | Execute |
| `python_eval` | Evaluate a Python expression | Execute |

### Web Operations

| Tool | Description | Permission Required |
|------|-------------|-------------------|
| `web_search` | Search the web using configured search engine | Network |
| `web_browse` | Browse a web page using Playwright | Network |
| `web_fetch` | Fetch the content of a URL | Network |
| `web_screenshot` | Take a screenshot of a web page | Network |

### Memory and Knowledge

| Tool | Description | Permission Required |
|------|-------------|-------------------|
| `memory_read` | Read from MEMORY.md | Read |
| `memory_write` | Write to MEMORY.md | Write |
| `memory_search` | Search through memory contents | Read |

### Session and Collaboration

| Tool | Description | Permission Required |
|------|-------------|-------------------|
| `session_spawn` | Spawn a new session | Execute |
| `session_send` | Send a message to another session | Execute |
| `canvas_draw` | Draw on the shared canvas | Write |

### Skill Management

| Tool | Description | Permission Required |
|------|-------------|-------------------|
| `skill_install` | Install a new skill | Write |
| `skill_list` | List installed skills | Read |
| `skill_execute` | Run a skill-specific action | Execute |

### Tool usage in conversations

You don't invoke tools directly — the agent decides which tools to use based on your request. However, you can influence tool usage:

```
# The agent will automatically use appropriate tools
> Read the file config.py and tell me what database it's configured to use
[Agent uses file_read tool]

# You can request specific tool behavior
> Search the web for the latest Python 3.13 release notes
[Agent uses web_search tool]

# You can ask the agent NOT to use certain tools
> Explain this concept without searching the web
[Agent answers from its training data]
```

---

## Voice Commands

ManusClaw v5.0.0 supports voice interaction through two modes: **voice wake** and **voice talk**. Voice commands allow hands-free operation of ManusClaw, making it ideal for use cases where typing isn't convenient.

### Voice Wake Mode

Voice wake mode listens for a wake word in the background. When detected, it activates ManusClaw for a single voice command.

```bash
# Start voice wake mode
manusclaw voice wake --wake-word "Hey ManusClaw"
```

Once active, say the wake word followed by your command:

```
User:    "Hey ManusClaw, what files are in the workspace?"
ManusClaw: [processes and responds]

User:    "Hey ManusClaw, run the test suite in the background"
ManusClaw: Background task started...
```

### Voice Talk Mode

Voice talk mode provides a continuous, back-and-forth voice conversation:

```bash
# Start voice talk mode
manusclaw voice talk
```

In voice talk mode, the conversation flows naturally:

```
User:      "What's the status of the code review task?"
ManusClaw: "The code review task is 75% complete. It found 3 security issues
            and 12 style violations so far. Would you like to see the details?"

User:      "Yes, show me the security issues"
ManusClaw: "Here are the 3 security issues found: ..."
```

### Voice Command Reference

The following voice commands are recognized by ManusClaw's voice system:

| Voice Command | Equivalent Action |
|---------------|-------------------|
| "new conversation" | `/new` |
| "save session" | `/save` |
| "show memory" | `/memory` |
| "show tools" | `/tools` |
| "show tasks" | `/tasks` |
| "clear screen" | `Ctrl+L` |
| "switch to build mode" | `/mode BUILD` |
| "switch to plan mode" | `/mode PLAN` |
| "compress context" | `/compress` |
| "show help" | `/help` |
| "exit" / "quit" | `/exit` |
| "search for ..." | Triggers web search |
| "run ... in background" | `/bg ...` |

### Voice Configuration

Voice features are configured in `config.toml`:

```toml
[voice]
# Wake word configuration
[wake]
enabled = true
wake_word = "Hey ManusClaw"
sensitivity = 0.7
language = "en-US"
audio_device = "default"

# Speech-to-text (ASR) configuration
[voice.asr]
provider = "openai"          # or "google", "whisper-local"
model = "whisper-1"
language = "en-US"

# Text-to-speech (TTS) configuration
[voice.tts]
enabled = true
provider = "openai"          # or "google", "piper-local"
voice = "en-US-Neural2-D"
speed = 1.0
output_device = "default"

# Voice activity detection
[voice.vad]
enabled = true
threshold = 0.5
silence_timeout = 2.0       # seconds of silence to end turn
```

### Voice with SSH Gateway

You can combine voice mode with the SSH gateway to interact with ManusClaw remotely:

```bash
# On the remote server
manusclaw-ssh start --port 2222

# On your local machine, connect via SSH and start voice talk
ssh -t -p 2222 user@server "manusclaw voice talk"
```

---

## SSH Gateway Usage

The SSH gateway (`manusclaw-ssh start`) exposes a full ManusClaw interactive shell over SSH, allowing remote access from any machine.

### Getting Started

```bash
# Start the SSH gateway
manusclaw-ssh start --port 2222

# Connect from any SSH client
ssh -p 2222 localhost
```

After connecting, you get a full ManusClaw shell:

```
$ ssh -p 2222 localhost
ManusClaw v5.0.0 — SSH Gateway
Provider: anthropic / Model: claude-sonnet-4-20250514
Workspace: /home/user/workspace

> Hello! I'm ready to help. What would you like to work on?
```

### Authentication Methods

#### Password Authentication

```bash
# Start with password authentication
manusclaw-ssh start --auth password --password-file /path/to/passwd
```

#### Public Key Authentication (recommended)

```bash
# Start with public key authentication
manusclaw-ssh start --auth pubkey --authorized-keys /path/to/authorized_keys

# Authorized keys file format is standard OpenSSH
# /path/to/authorized_keys:
# ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQD... user@laptop
```

#### API Key Authentication

```bash
# Start with API key authentication (for programmatic access)
manusclaw-ssh start --auth apikey --api-key-file /path/to/apikeys
```

### SSH Gateway Configuration

```toml
[ssh]
enabled = false
host = "0.0.0.0"
port = 2222
auth = "pubkey"
host_key = "~/.manusclaw/ssh_host_ed25519_key"
authorized_keys = "~/.manusclaw/authorized_keys"
max_sessions = 10
session_timeout = 3600        # 1 hour
log_connections = true
log_dir = "~/.manusclaw/ssh-logs"
allowed_users = ["alice", "bob"]
banner = "ManusClaw v5.0.0 — Unauthorized access is prohibited."
```

### SSH Security Best Practices

1. **Always use public key authentication** in production environments.
2. **Restrict allowed users** with `--allowed-users` to limit who can connect.
3. **Set a session timeout** to automatically disconnect idle sessions.
4. **Enable connection logging** to audit SSH access.
5. **Use a firewall** to restrict SSH access to trusted IP ranges.
6. **Rotate host keys** periodically with `manusclaw-ssh rotate-keys`.

### Programmatic SSH Usage

The SSH gateway can be used programmatically for automation:

```bash
# Run a single command via SSH
ssh -p 2222 localhost "manusclaw --no-color 'What files are in the workspace?'"

# Run a script that sends multiple commands
cat << 'EOF' | ssh -p 2222 localhost
/memory
/workspace --tree
/bg Run the full test suite
EOF
```

### SSH with Config Profiles

```bash
# Start the gateway with a specific profile
manusclaw-ssh start --profile production

# Each SSH session inherits the profile's configuration
ssh -p 2222 localhost
# Inside: ManusClaw starts with production profile settings
```

---

## Webhook Management

Webhooks allow external systems to trigger ManusClaw tasks, receive event notifications, and integrate ManusClaw into your CI/CD pipelines and automation workflows.

### How Webhooks Work

1. You register a webhook endpoint with ManusClaw (`manusclaw-webhook create`).
2. External systems send HTTP POST requests to the endpoint (e.g., when a GitHub PR is opened).
3. ManusClaw receives the payload and executes the configured action.
4. Results can be sent back to external systems via channels.

### Quick Start

```bash
# Register a webhook for GitHub push events
manusclaw-webhook create \
  --name github-push \
  --url /webhooks/github \
  --secret "my-github-webhook-secret" \
  --events "push" \
  --action "Review the pushed code changes for bugs and style issues"

# Register a webhook for deployment events
manusclaw-webhook create \
  --name deploy-webhook \
  --url /webhooks/deploy \
  --secret "deploy-secret-123" \
  --events "deploy" \
  --action "Run post-deployment verification tests" \
  --profile production
```

### Webhook Payload Processing

When a webhook receives a payload, ManusClaw processes it as follows:

1. **Validation** — The payload is validated against the registered secret (HMAC signature).
2. **Event routing** — The event type is matched against the webhook's registered events.
3. **Action execution** — The configured action is executed with the payload injected into the agent's context.
4. **Response** — The result is returned to the caller and optionally broadcast via channels.

### Webhook Payload Templates

You can use template variables in webhook actions to reference payload data:

```bash
# Action with template variables
manusclaw-webhook create \
  --name github-pr \
  --url /webhooks/github \
  --secret "github-secret" \
  --events "pull_request" \
  --action "Review this pull request from {{.author}}: {{.title}}. The changes are in branch {{.branch}}"
```

### CI/CD Integration Examples

#### GitHub Actions

```yaml
# .github/workflows/review.yml
name: AI Code Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger ManusClaw review
        run: |
          curl -X POST http://manusclaw-server:8000/webhooks/github \
            -H "Content-Type: application/json" \
            -H "X-Webhook-Secret: ${{ secrets.MANUSCLAW_WEBHOOK_SECRET }}" \
            -d '{
              "event": "pull_request",
              "author": "${{ github.actor }}",
              "title": "${{ github.event.pull_request.title }}",
              "branch": "${{ github.event.pull_request.head.ref }}",
              "body": "${{ github.event.pull_request.body }}"
            }'
```

#### GitLab CI

```yaml
# .gitlab-ci.yml
review:
  stage: review
  script:
    - |
      curl -X POST http://manusclaw-server:8000/webhooks/gitlab \
        -H "Content-Type: application/json" \
        -H "X-Webhook-Secret: $MANUSCLAW_WEBHOOK_SECRET" \
        -d '{
          "event": "merge_request",
          "author": "$GITLAB_USER_LOGIN",
          "title": "$CI_MERGE_REQUEST_TITLE"
        }'
```

#### Jenkins Pipeline

```groovy
// Jenkinsfile
pipeline {
  agent any
  stages {
    stage('AI Review') {
      steps {
        sh '''
          curl -X POST http://manusclaw-server:8000/webhooks/jenkins \
            -H "Content-Type: application/json" \
            -H "X-Webhook-Secret: ${MANUSCLAW_SECRET}" \
            -d '{"event": "build", "branch": "${BRANCH_NAME}"}'
        '''
      }
    }
  }
}
```

### Webhook Security

- **Always use secrets** to validate incoming webhook payloads.
- **Use HTTPS** in production to encrypt webhook traffic.
- **Rotate secrets** periodically: `manusclaw-webhook rotate-secret <name>`.
- **Restrict source IPs** using a firewall to prevent unauthorized webhook calls.
- **Log all deliveries** for auditing: `manusclaw-webhook logs <name>`.

### Webhook Delivery and Retries

```toml
[webhooks]
enabled = true
secret_rotation_days = 90
max_retries = 5
retry_delay = 30            # seconds between retries
timeout = 30               # request timeout in seconds
log_deliveries = true
log_retention_days = 30
```

---

## Session Management CLI

The `manusclaw-sessions` command provides full session management from the command line, enabling scripting and automation workflows.

### Common Workflows

#### Listing and inspecting sessions

```bash
# List all sessions
manusclaw-sessions list

# List with verbose output (messages, tokens, duration)
manusclaw-sessions list --verbose

# Filter sessions by name pattern
manusclaw-sessions list --filter "api-*"

# Show detailed info about a session
manusclaw-sessions info session_abc123

# Show session statistics
manusclaw-sessions stats
```

#### Sending messages to sessions

```bash
# Send a fire-and-forget message
manusclaw-sessions send session_abc123 "Check if the deployment succeeded"

# Send a message and wait for the response
manusclaw-sessions send session_abc123 "Summarize our progress" --wait

# Send a message and pipe the response to a file
manusclaw-sessions send session_abc123 "Generate a changelog from git log" --wait > changelog.md
```

#### Spawning and managing sessions

```bash
# Spawn a new session with a name
manusclaw-sessions spawn --name "automated-review"

# Spawn with a specific workspace and profile
manusclaw-sessions spawn \
  --name "security-scan" \
  --workspace /opt/production \
  --profile security

# Spawn with an initial prompt
manusclaw-sessions spawn \
  --name "morning-report" \
  --prompt "Generate a summary of all activity in the workspace since yesterday"

# Export a session for backup
manusclaw-sessions export session_abc123 --output backup/session_abc123.json

# Import a session from backup
manusclaw-sessions import backup/session_abc123.json

# Delete a session
manusclaw-sessions delete session_abc123

# Prune old sessions
manusclaw-sessions prune --older-than 30d
```

#### Automation Example

```bash
#!/bin/bash
# Automated code review pipeline using manusclaw-sessions

# Spawn a new session for the review
SESSION_ID=$(manusclaw-sessions spawn --name "auto-review-$BUILD_ID" --output-id)

# Send the code diff for review
manusclaw-sessions send "$SESSION_ID" "Review this code change: $(git diff origin/main...HEAD)" --wait > review_output.md

# Check the review output
if rg -i "critical|security|vulnerability" review_output.md; then
  echo "CRITICAL issues found. Failing the build."
  exit 1
fi

# Clean up the session
manusclaw-sessions delete "$SESSION_ID"
```

---

## Cron Job Management

Cron jobs allow you to schedule recurring ManusClaw tasks using standard cron expression syntax.

### Full Command Reference

```bash
# Start the cron daemon
manusclaw-cron

# Start the daemon with a specific profile
manusclaw-cron --profile production

# Start in the background (daemon mode)
manusclaw-cron --daemon

# Start with a custom config file
manusclaw-cron --config /path/to/custom-cron.toml
```

### Managing Scheduled Tasks

```bash
# List all scheduled tasks
manusclaw-cron --list

# Add a task with a cron expression
manusclaw-cron --add "0 9 * * 1" "Summarize the weekly meeting notes"

# Add a task with a name for easy management
manusclaw-cron --add --name "daily-security-scan" "0 2 * * *" "Run a security scan of the workspace"

# Add a task with a specific profile
manusclaw-cron --add --name "prod-health-check" --profile production "*/5 * * * *" "Check production server health"

# Remove a task by name
manusclaw-cron --remove --name "daily-security-scan"

# Remove a task by ID
manusclaw-cron --remove cron_001

# Pause all tasks
manusclaw-cron --pause

# Resume all tasks
manusclaw-cron --resume

# Pause a specific task
manusclaw-cron --pause --name "daily-security-scan"

# Resume a specific task
manusclaw-cron --resume --name "daily-security-scan"
```

### Task History and Logs

```bash
# Show execution history for all tasks
manusclaw-cron --history

# Show execution history for a specific task
manusclaw-cron --history --name "daily-security-scan"

# Show the last N executions
manusclaw-cron --history --limit 10

# Show execution output
manusclaw-cron --history --name "daily-security-scan" --output
```

### Import and Export

```bash
# Export the current schedule to a YAML file
manusclaw-cron --export schedule.yaml

# Import a schedule from a YAML file
manusclaw-cron --import schedule.yaml

# Validate a schedule file without importing
manusclaw-cron --import schedule.yaml --dry-run
```

### Chained Cron Jobs

Cron jobs can be chained so that the output of one job becomes the input of the next:

```bash
# Create a chain of tasks
manusclaw-cron --add --name "step1-scan" --chain "pipeline-1" "0 6 * * *" "Scan the codebase for TODO comments"
manusclaw-cron --add --name "step2-prioritize" --chain "pipeline-1" --after "step1-scan" "0 7 * * *" "Prioritize the TODOs by severity"
manusclaw-cron --add --name "step3-report" --chain "pipeline-1" --after "step2-prioritize" "0 8 * * *" "Generate a prioritized TODO report and send to Slack"
```

### Cron Job Configuration

```toml
[cron]
enabled = true
timezone = "UTC"
max_concurrent = 3
log_executions = true
log_dir = "~/.manusclaw/cron-logs"
on_failure = "retry"           # "retry", "skip", "notify"
max_retries = 3
retry_delay = 60
notification_channel = "slack-primary"
```

---

## Channel Management

Channels are named message streams that connect ManusClaw to external communication services. They allow ManusClaw to send notifications, reports, and results to platforms like Slack, Discord, email, and more.

### Channel Configuration

Channels can be configured in `config.toml` or via the `manusclaw-channels` CLI:

```toml
[[channels]]
name = "slack-primary"
type = "slack"
webhook_url = "${SLACK_WEBHOOK_URL}"
enabled = true
events = ["task.complete", "task.fail", "error", "cron.*"]

[[channels]]
name = "discord-alerts"
type = "discord"
webhook_url = "${DISCORD_WEBHOOK_URL}"
enabled = true
events = ["error", "cron.fail"]

[[channels]]
name = "email-reports"
type = "email"
smtp_host = "smtp.gmail.com"
smtp_port = 587
smtp_user = "manusclaw@example.com"
smtp_password = "${EMAIL_APP_PASSWORD}"
recipients = ["team@example.com"]
events = ["cron.complete", "report.*"]
```

### Event Types

Channels subscribe to specific event types. When an event fires, ManusClaw sends a formatted message to all channels subscribed to that event.

| Event Pattern | Fires When |
|---------------|------------|
| `task.complete` | Any background task completes successfully |
| `task.fail` | Any background task fails |
| `task.start` | Any background task starts |
| `error` | An error occurs in the ManusClaw agent |
| `cron.complete` | A cron job execution completes |
| `cron.fail` | A cron job execution fails |
| `cron.*` | Any cron-related event |
| `webhook.*` | Any webhook event |
| `session.start` | A new session is started |
| `report.*` | Any report is generated |
| `*` | All events |

### Channel Message Formatting

Each channel type automatically formats messages appropriately:

- **Slack:** Uses Slack Blocks API with rich formatting, code blocks, and action buttons.
- **Discord:** Uses Discord embed objects with fields and color coding.
- **Email:** Sends HTML-formatted emails with styled tables and code blocks.
- **Webhook:** Sends JSON payloads with structured event data.

### Advanced Channel Usage

```bash
# Create a channel with event filtering
manusclaw-channels create \
  --name "critical-alerts" \
  --type pagerduty \
  --events "error,cron.fail,webhook.fail" \
  --routing-key "${PAGERDUTY_ROUTING_KEY}"

# Create a channel with message templates
manusclaw-channels create \
  --name "custom-webhook" \
  --type webhook \
  --url https://api.mycompany.com/events \
  --template '{"event": "{{.Event}}", "message": "{{.Message}}", "timestamp": "{{.Timestamp}}"}'

# Test a channel with a sample event
manusclaw-channels test slack-primary --event "task.complete"

# Show delivery logs for a channel
manusclaw-channels logs slack-primary --limit 50
```

### Conditional Channel Routing

Channels can route messages based on conditions:

```toml
[[channels]]
name = "security-alerts"
type = "slack"
webhook_url = "${SLACK_SECURITY_WEBHOOK}"
events = ["error"]
conditions = """
  .Message contains "security" or
  .Message contains "vulnerability" or
  .Message contains "CVE-"
"""
```

---

## Server Endpoints Reference

The `manusclaw-server` exposes the following endpoints. All authenticated endpoints require the `Authorization: Bearer <api-key>` header unless the server is running without `--api-key`.

### REST Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/healthz` | GET | Health check. Returns server status, version, and uptime. |
| `/chat` | POST | Send a message to an active session and receive a response. |
| `/canvas` | GET/POST | Access the shared canvas for drawing and visualization. |
| `/multi-agent` | POST | Submit tasks to the multi-agent orchestrator. |
| `/webhooks/*` | POST | Receive webhook payloads from external systems. |
| `/sessions` | GET | List all active and saved sessions. |
| `/sessions/{id}` | GET | Get details of a specific session. |
| `/sessions/{id}/history` | GET | Get conversation history for a session. |
| `/sessions/{id}/send` | POST | Send a message to a session. |
| `/sessions/{id}/branch` | POST | Create a branch from a session. |
| `/tasks` | GET | List all tasks (running, completed, failed). |
| `/tasks/{id}` | GET | Get details of a specific task. |
| `/tasks/{id}/cancel` | POST | Cancel a running task. |
| `/tasks/{id}/output` | GET | Get output from a completed task. |
| `/skills` | GET | List installed and available skills. |
| `/skills/{name}/install` | POST | Install a skill. |
| `/config` | GET | Get the current server configuration. |

### WebSocket Endpoints

| Endpoint | Description |
|----------|-------------|
| `/ws/canvas/{session_id}` | Real-time canvas updates for a session. Streams drawing commands, annotations, and visual content. |
| `/ws/chat/{session_id}` | Real-time chat streaming for a session. Streams agent responses token by token. |

### Endpoint Examples

#### Health Check

```bash
curl http://localhost:8000/healthz
```

Response:
```json
{
  "status": "ok",
  "version": "5.0.0",
  "uptime": 3600,
  "active_sessions": 3,
  "running_tasks": 2
}
```

#### Chat Endpoint

```bash
curl -X POST http://localhost:8000/chat \
  -H "Authorization: Bearer my-secret-key" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "What files are in the workspace?",
    "session_id": "session_abc123",
    "mode": "PLAN"
  }'
```

Response:
```json
{
  "session_id": "session_abc123",
  "response": "Here are the files in the workspace:\n...",
  "tokens_used": 1234,
  "model": "claude-sonnet-4-20250514",
  "duration_ms": 2340
}
```

#### WebSocket Chat Connection

```javascript
// Browser-based WebSocket client
const ws = new WebSocket('ws://localhost:8000/ws/chat/session_abc123');

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.type === 'token') {
    process.stdout.write(data.content);  // Stream tokens
  } else if (data.type === 'done') {
    console.log('\n--- Response complete ---');
  }
};

ws.onopen = () => {
  ws.send(JSON.stringify({
    message: "Explain this code: main.py",
    mode: "PLAN"
  }));
};
```

#### Canvas WebSocket

```javascript
// Connect to a session's canvas
const ws = new WebSocket('ws://localhost:8000/ws/canvas/session_abc123');

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.type === 'draw') {
    renderCanvasCommand(data.command);
  }
};

// Send a canvas command
ws.send(JSON.stringify({
  type: 'draw',
  command: 'rectangle',
  params: { x: 10, y: 10, width: 100, height: 50 }
}));
```

#### Multi-Agent Endpoint

```bash
curl -X POST http://localhost:8000/multi-agent \
  -H "Authorization: Bearer my-secret-key" \
  -H "Content-Type: application/json" \
  -d '{
    "task": "Research the best practices for microservice architecture and implement them",
    "agents": 3,
    "strategy": "pipeline",
    "roles": ["researcher", "architect", "implementer"]
  }'
```

#### Webhook Endpoint

```bash
# External system sends a webhook payload
curl -X POST http://localhost:8000/webhooks/github \
  -H "Content-Type: application/json" \
  -H "X-Webhook-Secret: my-webhook-secret" \
  -d '{
    "event": "push",
    "author": "alice",
    "branch": "main",
    "commits": ["Fix authentication bug", "Update dependencies"]
  }'
```

---

## Advanced Usage Patterns

### Pattern 1: Iterative development with session branching

Use ManusClaw's branching feature to explore multiple approaches in parallel:

```
> Read the current authentication module and suggest improvements
[Agent suggests improvements]

> /branch "jwt-approach"
[On branch: jwt-approach]
> Implement the JWT-based approach
[Agent implements]

> /branch --switch main
> /branch "session-approach"
[On branch: session-approach]
> Implement the session-based approach instead
[Agent implements]

> /branch --list
> Compare both approaches and recommend the better one
> /branch --merge jwt-approach
```

### Pattern 2: Multi-agent pipeline

Set up a multi-agent pipeline for complex workflows:

```bash
# Start multi-agent with a pipeline strategy
manusclaw-multi --agents 3 --strategy pipeline --roles "researcher,coder,tester"

# Or configure it in config.toml
[multi_agent]
strategy = "pipeline"
agents = 3
roles = [
  { name = "researcher", model = "claude-sonnet-4-20250514", skills = ["web-search"] },
  { name = "coder", model = "gpt-4o", skills = ["web-dev"] },
  { name = "tester", model = "claude-sonnet-4-20250514", skills = ["security"] }
]
```

### Pattern 3: CI/CD with webhooks and cron

Combine webhooks and cron for a fully automated development pipeline:

```bash
# Register a webhook for PR events
manusclaw-webhook create \
  --name github-pr-review \
  --url /webhooks/github \
  --secret "$GITHUB_SECRET" \
  --action "Review this PR and provide feedback"

# Schedule daily health checks
manusclaw-cron --add --name "daily-health" --profile production "0 8 * * *" \
  "Check production server health and report any issues"

# Schedule weekly reports
manusclaw-cron --add --name "weekly-summary" "0 9 * * 5" \
  "Generate a weekly summary of all changes and send to Slack"
```

### Pattern 4: Voice-controlled development

Use voice commands for hands-free development:

```bash
# Start voice talk mode with a production profile
manusclaw voice talk --profile production --noise-cancel

# Then use voice commands:
User: "Hey ManusClaw, run the test suite in the background"
User: "What were the test results?"
User: "Fix the failing tests in the auth module"
User: "Commit these changes with a descriptive message"
User: "Deploy to staging"
```

### Pattern 5: Remote access with SSH + profiles

Access ManusClaw from anywhere using the SSH gateway with different profiles for different environments:

```bash
# On the server
manusclaw-ssh start --port 2222 --auth pubkey

# From your laptop - development work
ssh -p 2222 server "manusclaw --profile development"

# From your laptop - production checks
ssh -p 2222 server "manusclaw --profile production 'Show me the latest error logs'"

# From CI/CD - automated task
ssh -p 2222 server "manusclaw-sessions spawn --name 'ci-review' --prompt 'Review the latest commits'"
```

### Pattern 6: Model failover for reliability

Configure failover chains to ensure ManusClaw keeps working even during provider outages:

```bash
# Start with failover enabled
manusclaw --failover

# The agent automatically switches providers on errors
# You can also use it with the server for high-availability
manusclaw-server --failover --profile production

# Monitor failover events
> /model
Active: openai/gpt-4o (switched at 14:32:05)
Failover chain: anthropic/claude-sonnet-4-20250514 → openai/gpt-4o → ollama/llama3
Recent failovers: 1 (anthropic rate_limit → openai)
```

### Pattern 7: Combining foreground and background tasks

Work on one task while another runs in the background:

```
> /bg Run the full test suite and create a coverage report
Background task started (ID: task_tests)

> Meanwhile, let's work on the API documentation. Start by reading the current docs...
[You work on docs while tests run in the background]

[Background] task_tests completed. Coverage: 72%. See: workspace/coverage-report.html
```

### Pattern 8: Cross-session collaboration

Use session spawning and messaging to have multiple ManusClaw instances working together:

```
> /sessions spawn --name "frontend-work" --prompt "Work on the React components for the dashboard"
Session spawned: session_frontend (name: frontend-work)

> /sessions spawn --name "backend-work" --prompt "Work on the REST API endpoints for the dashboard"
Session spawned: session_backend (name: backend-work)

> Let's work on the integration tests while both sessions work in parallel
...

> /sessions send session_frontend "The API endpoints have changed. Here are the new routes..."
> /sessions history session_frontend --limit 5
```

### Pattern 9: Context compression for long sessions

For extended development sessions, periodically compress context to stay within token limits:

```
> [Long conversation about architecture decisions and code changes...]

> /compress
Context compressed.
  Before: 89,450 tokens (44.7% of budget)
  After:  18,200 tokens (9.1% of budget)
  Key points preserved.

> [Continue working with fresh context...]
```

---

## Summary

ManusClaw v5.0.0 provides a comprehensive toolkit for AI-assisted development:

| Feature | Entry Point / Command | Key Use Case |
|---------|---------------------|--------------|
| Interactive chat | `manusclaw` | Day-to-day coding assistance |
| HTTP API | `manusclaw-server` | Integration into apps and pipelines |
| Multi-agent | `manusclaw-multi` | Complex parallel workflows |
| Cron scheduling | `manusclaw-cron` | Recurring automated tasks |
| Session management | `manusclaw-sessions` | CLI-based session control |
| Channel notifications | `manusclaw-channels` | Alerts to Slack, Discord, email |
| Webhook triggers | `manusclaw-webhook` | CI/CD and external integrations |
| Voice interaction | `manusclaw voice wake/talk` | Hands-free operation |
| Remote access | `manusclaw-ssh start` | SSH-based remote sessions |
| Model failover | `--failover` flag | High-availability LLM access |
| Config profiles | `--profile` flag | Environment-specific settings |

For installation instructions, see [installation.md](installation.md). For configuration details, see [configuration.md](configuration.md). For troubleshooting, see [troubleshooting.md](troubleshooting.md).
