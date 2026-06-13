# Usage Guide — ManusClaw v5.1.0

This guide covers everything you need to know about using ManusClaw v5.1.0 on a day-to-day basis. From launching your first session to advanced features like multi-agent orchestration, voice commands, SSH gateways, webhook management, cron scheduling, model failover, config profiles, hooks, secrets, parallel execution, and the skills system — every feature is explained with practical examples.

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
  - [manusclaw-parallel — Parallel Executor (v5.1)](#manusclaw-parallel--parallel-executor-v51)
  - [manusclaw-migrate — Migration System (v5.1)](#manusclaw-migrate--migration-system-v51)
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
- [Skills System](#skills-system)
- [Security & RBAC (v5.1)](#security--rbac-v51)
- [Hooks (v5.1)](#hooks-v51)
- [Secrets Management (v5.1)](#secrets-management-v51)
- [Context & Conversations (v5.1)](#context--conversations-v51)
- [File Store (v5.1)](#file-store-v51)
- [Git Providers (v5.1)](#git-providers-v51)
- [Observability (v5.1)](#observability-v51)

---

## Entry Point Commands

### manusclaw — The Main Agent

The primary command for running the ManusClaw agent. Supports three modes:

```bash
# Interactive REPL mode (default)
manusclaw

# Single-shot mode
manusclaw "Create a Python REST API with FastAPI"

# With specific provider and model
manusclaw --provider groq --model llama-3.3-70b-versatile "Explain recursion"

# With a config profile
manusclaw --profile production "Analyze the sales data"

# Validate configuration
manusclaw --validate-config

# List available features
manusclaw --features

# Show version
manusclaw --version
```

### manusclaw-server — HTTP API Server

Run ManusClaw as an HTTP/WebSocket server for API access, WebChat, Canvas, and channel adapters.

```bash
# Start the server with default settings
manusclaw-server

# Custom host and port
manusclaw-server --host 0.0.0.0 --port 8765

# With API key authentication
MANUSCLAW_SERVER_API_KEY=your-key manusclaw-server

# With collaborative canvas (v5.1)
manusclaw-server --canvas-mode collaborative

# With Prometheus metrics (v5.1)
manusclaw-server --metrics-port 9090
```

**Server API endpoints:**

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health` | GET | Health check |
| `/api/chat` | POST | Send a chat message |
| `/api/sessions` | GET | List sessions |
| `/api/sessions/{id}` | GET | Get session details |
| `/ws/{session_id}` | WebSocket | WebSocket chat |
| `/ws/canvas/{session_id}` | WebSocket | Canvas WebSocket |
| `/metrics` | GET | Prometheus metrics (v5.1) |
| `/webhooks/{id}` | POST | Webhook receiver |

### manusclaw-multi — Multi-Agent Orchestrator

Run multiple agents in parallel or in sequence.

```bash
# Run two agents in parallel
manusclaw-multi --agent1 "Analyze the frontend code" --agent2 "Analyze the backend code"

# Sequential pipeline
manusclaw-multi --pipeline "Research the topic" "Write a blog post" "Review and edit"
```

### manusclaw-cron — Scheduled Task Runner

```bash
# Start the cron scheduler
manusclaw-cron start

# List scheduled jobs
manusclaw-cron list

# Add a job
manusclaw-cron add --name "Daily Report" --schedule "0 9 * * *" \
  --prompt "Generate the daily summary report"

# Run a job immediately
manusclaw-cron run --name "Daily Report"

# Remove a job
manusclaw-cron remove --name "Daily Report"
```

### manusclaw-sessions — Session Management CLI

```bash
# List all sessions
manusclaw-sessions list

# Show session history
manusclaw-sessions history --session abc123

# Show session history with tool calls
manusclaw-sessions history --session abc123 --tool-calls

# Send a message to a running session
manusclaw-sessions send --session abc123 --message "Continue the task"

# Spawn a new session
manusclaw-sessions spawn --prompt "Analyze the data"

# Delete a session
manusclaw-sessions delete --session abc123 --force

# Export session as JSON
manusclaw-sessions export --session abc123 --output session.json
```

### manusclaw-channels — Channel Management CLI

```bash
# Start a specific channel
manusclaw-channels start telegram
manusclaw-channels start discord
manusclaw-channels start slack
manusclaw-channels start whatsapp
manusclaw-channels start signal
manusclaw-channels start matrix
manusclaw-channels start irc
manusclaw-channels start twitch
manusclaw-channels start teams        # v5.1: Full support
manusclaw-channels start google_chat  # v5.1: Full support
manusclaw-channels start line         # v5.1: New channel

# Start multiple channels
manusclaw-channels start telegram discord slack

# List active channels
manusclaw-channels list

# Stop a channel
manusclaw-channels stop telegram

# Show channel status
manusclaw-channels status
```

### manusclaw-webhook — Webhook Management CLI

```bash
# Create a webhook
manusclaw-webhook create \
  --url "/webhooks/github-push" \
  --secret "my-webhook-secret" \
  --prompt "Analyze push: {{payload.head_commit.message}}"

# List all webhooks
manusclaw-webhook list

# Delete a webhook
manusclaw-webhook delete --id github-push

# Get HMAC signature for testing
manusclaw-webhook sign --id github-push \
  --payload '{"head_commit": {"message": "test commit"}}'

# Test a webhook
manusclaw-webhook test --id github-push \
  --payload '{"head_commit": {"message": "test commit"}}'
```

### manusclaw-parallel — Parallel Executor (v5.1)

Execute multiple agent tasks concurrently.

```bash
# Run tasks in parallel
manusclaw-parallel run \
  --task "Analyze the sales data" \
  --task "Generate a marketing report" \
  --task "Check for anomalies"

# With dependencies
manusclaw-parallel run \
  --task "Fetch data from API" --id fetch \
  --task "Process the data" --id process --depends-on fetch \
  --task "Generate report" --id report --depends-on process

# Using Ray for distributed execution
manusclaw-parallel run --mode ray --workers 8 \
  --task "Analyze dataset 1" \
  --task "Analyze dataset 2"

# Check parallel executor status
manusclaw-parallel status

# Cancel running tasks
manusclaw-parallel cancel --all
```

### manusclaw-migrate — Migration System (v5.1)

Manage database and configuration migrations.

```bash
# Check migration status
manusclaw-migrate status

# Run pending migrations
manusclaw-migrate run

# Migrate from specific version
manusclaw-migrate --from 5.0.0 --to 5.1.0

# Rollback last migration
manusclaw-migrate rollback

# Rollback to specific version
manusclaw-migrate rollback --to 5.0.0

# Create a new migration
manusclaw-migrate create --name "add_conversation_tables"

# List all migrations
manusclaw-migrate list
```

### manusclaw voice wake — Voice Wake Mode

```bash
# Start wake word detection
manusclaw voice wake --start --word "hey manus" --sensitivity 0.7

# With specific backend
manusclaw voice wake --start --backend porcupine

# Stop wake word detection
manusclaw voice wake --stop
```

### manusclaw voice talk — Voice Talk Mode

```bash
# Start talk mode
manusclaw voice talk --start

# With streaming STT (v5.1)
manusclaw voice talk --start --streaming

# With specific TTS provider
manusclaw voice talk --start --tts elevenlabs

# Stop talk mode
manusclaw voice talk --stop
```

### manusclaw-ssh start — SSH Gateway

```bash
# Start the SSH gateway
export MANUSCLAW_SSH_ENABLED=true
export MANUSCLAW_SSH_PORT=2222
manusclaw-ssh start

# Connect via SSH
ssh -p 2222 admin@your-server

# Available commands inside SSH session:
# status, restart, logs, agent <prompt>, channels list,
# cron list, config get <key>, config set <key> <value>,
# help, exit
```

---

## CLI Flags Reference

### Global Flags

| Flag | Description | Default |
|------|-------------|---------|
| `--provider` | LLM provider | From config |
| `--model` | LLM model | From config |
| `--profile` | Config profile | `default` |
| `--temperature` | LLM temperature | From config |
| `--max-tokens` | Maximum output tokens | From config |
| `--validate-config` | Validate configuration and exit | — |
| `--features` | List available features and exit | — |
| `--version` | Show version and exit | — |
| `--help` | Show help and exit | — |

### Server Flags

| Flag | Description | Default |
|------|-------------|---------|
| `--host` | Server bind address | `0.0.0.0` |
| `--port` | Server bind port | `8765` |
| `--canvas-mode` | Canvas mode (`single` or `collaborative`) | `single` |
| `--metrics-port` | Prometheus metrics port | `9090` |

---

## Config Profiles

Config profiles let you switch between different configurations easily:

```bash
# List profiles
manusclaw profile list

# Create a profile from current config
manusclaw profile create production

# Create from template
manusclaw profile create development --template minimal

# Switch to a profile
manusclaw profile switch production

# Show active profile
manusclaw profile active

# Use a profile for a single command
manusclaw --profile quality "Complex analysis task"
```

### Profile Templates

| Template | Description |
|----------|-------------|
| `minimal` | Bare minimum configuration |
| `development` | Debug logging, cheap models, loose security |
| `production` | Warning logging, quality models, strict security |
| `enterprise` | Full security, observability, Vault, audit logging |

---

## Model Failover

When the primary LLM fails, ManusClaw automatically falls back to the next provider:

```bash
# View current failover chain
manusclaw config get model_profiles.default

# The agent automatically tries providers in priority order:
# 1. groq/llama-3.3-70b-versatile (priority 1)
# 2. openai/gpt-4o (priority 2)
# 3. anthropic/claude-sonnet-4-20250514 (priority 3)
```

Failures trigger exponential cooldown (30s → 60s → 120s → ...) per provider.

---

## Interactive Shell

When you run `manusclaw` without arguments, you enter the interactive REPL:

```
🐾 ManusClaw v5.1.0 — Autonomous AI Agent

Provider: openai/gpt-4o | Profile: default | Workspace: ./workspace

> Help me create a Python script that fetches weather data
```

### Interactive Features

- **Rich formatting** — Markdown rendering, syntax highlighting, progress bars
- **Tab completion** — Complete commands and file paths
- **Command history** — Arrow keys to navigate previous inputs
- **Multi-line input** — Use `\` at end of line for continuation

---

## Single-Shot Mode

Run a single task and exit:

```bash
# Basic
manusclaw "What is the capital of France?"

# Piped input
echo "Error: connection refused at port 8080" | manusclaw "What does this error mean?"

# File input
manusclaw "Summarize this document" < report.txt

# With provider override
manusclaw --provider groq "Quick question"

# Capture output
manusclaw "Write a Python function" > function.py
```

---

## Slash Commands Reference

In the interactive shell, use slash commands for special operations:

| Command | Description |
|---------|-------------|
| `/help` | Show available commands |
| `/status` | Show agent status and current session info |
| `/profile <name>` | Switch config profile |
| `/provider <provider> [model]` | Switch LLM provider |
| `/model <model>` | Switch LLM model |
| `/save` | Save current session |
| `/export [format]` | Export session (json, markdown, html) |
| `/clear` | Clear conversation history |
| `/context` | Show current context usage |
| `/context compress` | Manually trigger context compression (v5.1) |
| `/context summary` | Show context summary (v5.1) |
| `/memory` | Show memory contents |
| `/memory save <text>` | Save text to long-term memory |
| `/memory search <query>` | Search long-term memory |
| `/skills` | List available skills |
| `/skills run <name>` | Run a skill |
| `/tasks` | List background tasks |
| `/sessions` | List all sessions |
| `/conversations` | List conversations (v5.1) |
| `/conversations search <query>` | Search conversations (v5.1) |
| `/files upload <path>` | Upload file to file store (v5.1) |
| `/files list` | List files in file store (v5.1) |
| `/secrets list` | List secret paths (v5.1) |
| `/hooks list` | List registered hooks (v5.1) |
| `/quit` or `/exit` | Exit the REPL |

---

## Background Task Execution

Run long-running tasks in the background:

```bash
# In the interactive shell:
> /background Analyze the entire codebase and generate documentation

# Or from CLI:
manusclaw-sessions spawn --prompt "Analyze the codebase" --background
```

### Monitoring Background Tasks

```bash
# List background tasks
manusclaw-sessions list --filter background

# Check task status
manusclaw-sessions history --session abc123 --last-message

# Cancel a task
manusclaw-sessions delete --session abc123 --force
```

---

## Task Queue Management

The task queue manages pending and running agent tasks:

```bash
# View queue status
manusclaw-sessions queue status

# Pause the queue
manusclaw-sessions queue pause

# Resume the queue
manusclaw-sessions queue resume

# Clear the queue
manusclaw-sessions queue clear
```

---

## Persistent Task System

Tasks can persist across sessions using the persistent task system:

```bash
# Create a persistent task
manusclaw-sessions spawn --prompt "Monitor the API endpoint" --persistent

# List persistent tasks
manusclaw-sessions list --filter persistent

# Resume a persistent task
manusclaw-sessions send --session abc123 --message "Check the status"
```

---

## Session Management

### Session Lifecycle

1. **Created** — New session is initialized
2. **Active** — Agent is processing a task
3. **Idle** — Waiting for user input
4. **Background** — Running in the background
5. **Completed** — Task finished successfully
6. **Failed** — Task encountered an error
7. **Archived** — Session data saved for later retrieval

### Session Operations

```bash
# Create a new session
manusclaw-sessions spawn --prompt "Analyze the data"

# List sessions with filters
manusclaw-sessions list
manusclaw-sessions list --filter active
manusclaw-sessions list --filter background
manusclaw-sessions list --limit 10

# Export session
manusclaw-sessions export --session abc123 --format json --output session.json
manusclaw-sessions export --session abc123 --format markdown --output session.md

# Resume a session
manusclaw-sessions send --session abc123 --message "Continue the task"

# Delete a session
manusclaw-sessions delete --session abc123 --force
```

---

## Skills System

Skills are reusable, parameterized task templates that the agent can invoke.

### Using Skills

```bash
# List available skills
manusclaw-skills list

# Run a skill
manusclaw-skills run code_review --path ./src/main.py

# In the interactive shell:
> /skills run code_review --path ./src/main.py
```

### Creating Custom Skills

Create a skill definition file in `~/.manusclaw/skills/`:

```yaml
# ~/.manusclaw/skills/code_review.yaml
name: code_review
description: "Review code for bugs, style issues, and improvements"
version: "1.0.0"

parameters:
  - name: path
    description: "Path to the file or directory to review"
    required: true
    type: string
  - name: focus
    description: "Focus area (security, performance, style, all)"
    required: false
    type: string
    default: "all"

prompt: |
  Review the code at {{path}} with focus on {{focus}}.
  Identify bugs, style issues, and potential improvements.
  Provide specific line references and suggestions.
```

---

## Security & RBAC (v5.1)

### Managing Roles

```bash
# List roles
manusclaw security roles list

# Create a role
manusclaw security roles create --name reviewer \
  --permissions "agent:execute,session:read,session:write"

# Update a role
manusclaw security roles update --name reviewer \
  --add-permissions "tools:use"

# Delete a role
manusclaw security roles delete --name reviewer
```

### Managing Users

```bash
# Assign a role to a user
manusclaw security users assign --user-id telegram_user_123 --role admin

# Remove a role from a user
manusclaw security users unassign --user-id telegram_user_123

# Show user permissions
manusclaw security users permissions --user-id telegram_user_123
```

### Rate Limiting

```bash
# Show current rate limit status
manusclaw security rate-limit status

# Reset rate limits for a user
manusclaw security rate-limit reset --user-id telegram_user_123
```

### Audit Logs

```bash
# View recent audit logs
manusclaw security audit-logs list --limit 50

# Filter by event type
manusclaw security audit-logs list --filter authentication

# Filter by user
manusclaw security audit-logs list --user-id telegram_user_123
```

---

## Hooks (v5.1)

### Managing Hooks

```bash
# List registered hooks
manusclaw hooks list

# Test a hook
manusclaw hooks test --name "log_request" --hook-type pre_execute

# Enable/disable a hook
manusclaw hooks enable --name "log_request"
manusclaw hooks disable --name "log_request"
```

### Hook Execution Flow

When a hook is configured:

1. Agent receives a task
2. **Pre-execute hooks** run (can abort the task)
3. Agent processes the task
4. **Post-execute hooks** run (notification, archiving)
5. If an error occurs, **on-error hooks** run

---

## Secrets Management (v5.1)

### Managing Secrets

```bash
# Store a secret
manusclaw secrets set openai/api_key "sk-proj-xxx"

# Retrieve a secret
manusclaw secrets get openai/api_key

# List secret paths (not values)
manusclaw secrets list

# Delete a secret
manusclaw secrets delete openai/api_key

# Rotate a secret
manusclaw secrets rotate openai/api_key

# Check secrets backend status
manusclaw secrets status
```

### Switching Backends

```bash
# Switch from env to Vault
manusclaw config set secrets.backend vault

# Verify Vault connection
manusclaw secrets status
```

---

## Context & Conversations (v5.1)

### Context Management

```bash
# Show current context usage
manusclaw context status

# Manually trigger compression
manusclaw context compress

# Show context summary
manusclaw context summary

# Clear context
manusclaw context clear
```

### Conversation Management

```bash
# List conversations
manusclaw conversations list

# Search conversations
manusclaw conversations search "deployment"

# Export a conversation
manusclaw conversations export --session abc123 --format markdown --output conversation.md

# Delete a conversation
manusclaw conversations delete --session abc123
```

---

## File Store (v5.1)

### File Store Commands

```bash
# Upload a file
manusclaw files upload report.pdf

# Upload with metadata
manusclaw files upload report.pdf --meta '{"category": "reports", "year": 2025}'

# Download a file
manusclaw files download report.pdf --output ./downloads/

# List stored files
manusclaw files list

# List with filters
manusclaw files list --prefix "reports/2025/"

# Delete a file
manusclaw files delete report.pdf

# Get file metadata
manusclaw files info report.pdf

# Check storage usage
manusclaw files usage
```

---

## Git Providers (v5.1)

### Git Provider Commands

```bash
# List repositories
manusclaw git repos list --provider github

# Read a file from a repo
manusclaw git file read --provider github --repo owner/repo --path README.md

# Create an issue
manusclaw git issue create --provider github --repo owner/repo \
  --title "Bug in authentication" --body "Description of the bug"

# Search code
manusclaw git search --provider github --repo owner/repo --query "auth middleware"
```

---

## Observability (v5.1)

### Checking Observability Status

```bash
# Show observability status
manusclaw observability status

# Show current metrics
manusclaw observability metrics

# Show recent traces
manusclaw observability traces --limit 20

# Show trace details
manusclaw observability trace --id trace-abc123

# Export metrics
manusclaw observability metrics-export --format prometheus --output metrics.txt
```

### Log Management

```bash
# View recent logs
manusclaw logs tail --lines 100

# View logs with level filter
manusclaw logs tail --level ERROR

# Search logs
manusclaw logs search "authentication failed"

# Export logs
manusclaw logs export --format json --output logs.json
```
