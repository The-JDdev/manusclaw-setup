# ManusClaw v5.1.0 — Feature Showcase & Usage Guide

> Complete guide for all features in ManusClaw v5.1.0, organized by category. Each section includes configuration, commands, and real-world usage examples.

---

## Table of Contents

### Core Agent Features
- [1. Autonomous Agent Execution](#1-autonomous-agent-execution)
- [2. Multi-Agent Routing](#2-multi-agent-routing)
- [3. Model Failover Profiles](#3-model-failover-profiles)
- [4. Credential Pool](#4-credential-pool)

### Communication & Channels
- [5. 15+ Messaging Channels](#5-15-messaging-channels)
- [6. SSH Remote Gateway](#6-ssh-remote-gateway)
- [7. Webhooks (Incoming)](#7-webhooks-incoming)
- [8. Gmail Pub/Sub Automation](#8-gmail-pubsub-automation)

### Voice & Canvas
- [9. Voice Wake & Talk Mode](#9-voice-wake--talk-mode)
- [10. Live Canvas (A2UI) v2](#10-live-canvas-a2ui-v2)

### Enterprise & Security (v5.1)
- [11. Security Configuration](#11-security-configuration)
- [12. Hooks System](#12-hooks-system)
- [13. Secrets Management](#13-secrets-management)

### Data & Storage (v5.1)
- [14. Context Management](#14-context-management)
- [15. Conversation Management](#15-conversation-management)
- [16. File Store](#16-file-store)

### Observability & Operations (v5.1)
- [17. Observability](#17-observability)
- [18. Parallel Executor](#18-parallel-executor)
- [19. Migration System](#19-migration-system)

### Integrations (v5.1)
- [20. Git Providers](#20-git-providers)
- [21. Integrations Plugin System](#21-integrations-plugin-system)

### Tools & Scheduling
- [22. 12+ LLM Providers](#22-12-llm-providers)
- [23. Enhanced Cron](#23-enhanced-cron)
- [24. Session Tools CLI](#24-session-tools-cli)
- [25. Sandbox Backends](#25-sandbox-backends)
- [26. Companion Apps](#26-companion-apps)

---

## 1. Autonomous Agent Execution

ManusClaw's core is an autonomous agent that plans, executes, and verifies tasks using a suite of built-in tools.

### How It Works

The agent follows a Plan → Execute → Verify loop:

1. **Plan** — Analyze the task and create a step-by-step plan
2. **Execute** — Use tools to carry out each step
3. **Verify** — Check that the result matches the intended outcome

### Built-in Tools

| Tool | Description |
|------|-------------|
| `file_read` | Read file contents |
| `file_write` | Write or create files |
| `file_edit` | Make targeted edits to existing files |
| `shell_execute` | Run shell commands |
| `web_search` | Search the web |
| `web_browse` | Browse and extract content from web pages |
| `code_execute` | Execute code in sandboxed environments |
| `ask_user` | Ask the user for clarification |
| `memory_save` | Save information to long-term memory |
| `memory_recall` | Recall information from long-term memory |

### Usage

```bash
# Interactive mode
manusclaw

# Single-shot mode
manusclaw "Create a Python REST API with FastAPI"

# With specific provider/model
manusclaw --provider groq --model llama-3.3-70b-versatile "Explain quantum computing"
```

---

## 2. Multi-Agent Routing

Route incoming messages to specialized agent types based on channel, user, or content patterns.

### Configuration

```yaml
agents:
  definitions:
    - name: manus
      class_path: app.agent.manus.Manus
    - name: data_analyst
      class_path: app.agent.data_analysis.DataAnalysisAgent
    - name: code_reviewer
      class_path: app.agent.code_review.CodeReviewAgent
    - name: devops
      class_path: app.agent.devops.DevOpsAgent

  routes:
    - pattern: "channel:telegram"
      agent: manus
      priority: 1
    - pattern: "channel:discord,#analytics"
      agent: data_analyst
      priority: 2
    - pattern: "channel:discord,#code-review"
      agent: code_reviewer
      priority: 3
    - pattern: "keyword:deploy,kubernetes"
      agent: devops
      priority: 4
```

### Route Pattern Types

| Pattern | Example | Matches |
|---------|---------|---------|
| Channel | `channel:telegram` | All Telegram messages |
| Channel + Topic | `channel:discord,#analytics` | Messages in specific channel/topic |
| User | `user_id:admin_user` | Messages from specific user |
| Keyword | `keyword:deploy,kubernetes` | Messages containing keywords |
| Regex | `regex:^bug:` | Messages matching regex pattern |

---

## 3. Model Failover Profiles

When the primary LLM provider fails or hits rate limits, ManusClaw automatically switches to the next provider in the failover chain.

### Configuration

```yaml
model_profiles:
  default:
    - provider: groq
      model: llama-3.3-70b-versatile
      priority: 1
    - provider: openai
      model: gpt-4o
      priority: 2
    - provider: anthropic
      model: claude-sonnet-4-20250514
      priority: 3
```

### Adaptive Cooldown

- Failed models enter exponential cooldown (starts at 30s, doubles each failure)
- Successful calls immediately reset the cooldown timer
- Per-session profiles allow different routing for different tasks

### Using Profiles

```bash
# Use the default profile
manusclaw "Analyze this data"

# Use a specific profile
manusclaw --profile fast "Quick question"
manusclaw --profile quality "Complex analysis"
manusclaw --profile local "Offline task"
```

---

## 4. Credential Pool

Rotate between multiple API keys to maximize throughput and avoid rate limits.

### Configuration

Keys are defined as numbered environment variables:

```bash
OPENAI_API_KEY_1=sk-proj-key-one
OPENAI_API_KEY_2=sk-proj-key-two
OPENAI_API_KEY_3=sk-proj-key-three

ANTHROPIC_API_KEY_1=sk-ant-key-one
ANTHROPIC_API_KEY_2=sk-ant-key-two
```

The pool automatically detects all numbered keys and rotates between them using round-robin with automatic cooldown on rate-limited keys.

### Combining with Model Failover

```yaml
model_profiles:
  default:
    - provider: openai       # Uses OPENAI_API_KEY_1, _2, _3 pool
      model: gpt-4o
      priority: 1
    - provider: anthropic   # Uses ANTHROPIC_API_KEY_1, _2 pool
      model: claude-sonnet-4-20250514
      priority: 2
```

---

## 5. 15+ Messaging Channels

### Channel Status Matrix

| # | Channel | Protocol | Status | Requirements |
|---|---------|----------|--------|-------------|
| 1 | **Telegram** | Bot API | ✅ Full | `TELEGRAM_BOT_TOKEN` |
| 2 | **Discord** | Gateway | ✅ Full | `DISCORD_BOT_TOKEN` |
| 3 | **Slack** | Web API | ✅ Full | `SLACK_BOT_TOKEN` |
| 4 | **WhatsApp** | Business Cloud | ✅ Full | `WHATSAPP_ACCESS_TOKEN`, `WHATSAPP_BUSINESS_PHONE_ID` |
| 5 | **Signal** | REST API | ✅ Full | `SIGNAL_CLI_REST_URL`, `SIGNAL_CLI_NUMBER` |
| 6 | **Microsoft Teams** | Bot Framework | ✅ Full | `MICROSOFT_APP_ID`, `MICROSOFT_APP_PASSWORD` |
| 7 | **Matrix** | Homeserver REST | ✅ Full | `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN`, `MATRIX_USER_ID` |
| 8 | **IRC** | Raw TCP | ✅ Full | `IRC_SERVER`, `IRC_PORT`, `IRC_NICK` |
| 9 | **Twitch** | IRC-over-TLS | ✅ Full | `TWITCH_BOT_TOKEN`, `TWITCH_CHANNEL` |
| 10 | **WebChat** | WebSocket | ✅ Full | None — built-in |
| 11 | **Google Chat** | Chat API v1 | ✅ Full | `GOOGLE_CHAT_SERVICE_ACCOUNT` |
| 12 | **LINE** | Messaging API | ✅ Full | `LINE_CHANNEL_SECRET`, `LINE_CHANNEL_ACCESS_TOKEN` |
| 13 | **Email** | SMTP/IMAP | 🔧 Stub | `EMAIL_SMTP_HOST`, `EMAIL_USER`, `EMAIL_PASS` |

### Starting Channels

```bash
# Start a specific channel
manusclaw-channels start telegram
manusclaw-channels start discord
manusclaw-channels start slack

# Start multiple channels
manusclaw-channels start telegram discord slack

# List active channels
manusclaw-channels list

# Stop a channel
manusclaw-channels stop telegram
```

### WhatsApp Setup

```bash
export WHATSAPP_ACCESS_TOKEN="EAAxxxxx"
export WHATSAPP_BUSINESS_PHONE_ID="1234567890"
manusclaw-channels start whatsapp
```

### Signal Setup

```bash
export SIGNAL_CLI_REST_URL=http://localhost:8080
export SIGNAL_CLI_NUMBER=+1234567890
manusclaw-channels start signal
```

### Microsoft Teams Setup (Full in v5.1)

Teams is now fully supported in v5.1.0:

```bash
export MICROSOFT_APP_ID="your-app-id"
export MICROSOFT_APP_PASSWORD="your-app-password"
manusclaw-channels start teams
```

### Google Chat Setup (Full in v5.1)

Google Chat is now fully supported in v5.1.0:

```bash
export GOOGLE_CHAT_SERVICE_ACCOUNT='{"type": "service_account", ...}'
manusclaw-channels start google_chat
```

### LINE Setup (New in v5.1)

LINE messaging is new in v5.1.0:

```bash
export LINE_CHANNEL_SECRET="your-channel-secret"
export LINE_CHANNEL_ACCESS_TOKEN="your-access-token"
manusclaw-channels start line
```

---

## 6. SSH Remote Gateway

Access ManusClaw remotely via SSH with public key authentication and a restricted command set.

### Setup

```bash
export MANUSCLAW_SSH_ENABLED=true
export MANUSCLAW_SSH_PORT=2222
export MANUSCLAW_SSH_AUTH_KEYS=~/.ssh/authorized_keys
manusclaw-ssh start
```

### Available Commands

| Command | Description |
|---------|-------------|
| `status` | System health check |
| `restart` | Restart the agent |
| `logs` | View recent logs |
| `agent <prompt>` | Send prompt to the running agent |
| `channels list` | Show active messaging channels |
| `cron list` | Show scheduled cron jobs |
| `config get <key>` | Get config value (v5.1) |
| `config set <key> <value>` | Set runtime config (v5.1) |
| `help` | Show available commands |
| `exit` | Disconnect |

### Security

- **Public key auth only** — no password authentication
- **Command whitelist** — only approved commands
- **Input validation** — rejects pipes, redirects, shell metacharacters
- **Command chaining blocked** — only one command per input

---

## 7. Webhooks (Incoming)

Create webhook endpoints that trigger agent actions when external services send events.

### Creating a Webhook

```bash
manusclaw-webhook create \
  --url "/webhooks/github-push" \
  --secret "my-webhook-secret" \
  --prompt "Analyze this GitHub push: {{payload.head_commit.message}}"

manusclaw-webhook list
manusclaw-webhook delete --id github-push
manusclaw-webhook sign --id github-push --payload '{"head_commit": {"message": "test"}}'
```

### Template Variables

Use `{{payload.field}}` syntax to interpolate webhook payload data into the agent prompt:

```json
{
  "head_commit": {
    "message": "Fix authentication bug",
    "author": {"name": "John"}
  }
}
```

Prompt template: `"Analyze push: {{payload.head_commit.message}} by {{payload.head_commit.author.name}}"`

### HMAC Verification

All webhooks use HMAC-SHA256 signature verification. Incoming requests must include a valid `X-Signature` header.

---

## 8. Gmail Pub/Sub Automation

Automate email processing using Google Cloud Pub/Sub push notifications.

### Setup

```bash
# Set up Google Cloud credentials
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json

# Create Pub/Sub topic
gcloud pubsub topics create projects/manusclaw-gmail-topic

# Enable in config
export GMAIL_WATCH_TOPIC_NAME=projects/manusclaw-gmail-topic
export GMAIL_AUTO_REPLY=true
```

The GmailWatcher receives push notifications, processes emails with the agent, and optionally sends auto-replies.

---

## 9. Voice Wake & Talk Mode

### Wake Word Detection

Three backends with automatic fallback:

| Backend | Priority | Requirements | Latency |
|---------|----------|-------------|---------|
| **Porcupine** | 1st | `PICOVOICE_API_KEY`, pyaudio | ~50ms |
| **Google STT** | 2nd | `speech_recognition` package | ~2s |
| **Streaming STT** (v5.1) | 3rd | WebSocket connection | ~200ms |
| **Stub** | Last | None | 30s simulation |

```bash
# With Porcupine (recommended)
export PICOVOICE_API_KEY=your-key
manusclaw voice wake --start --word "hey manus" --sensitivity 0.7
```

### Talk Mode

Continuous voice conversation with STT and TTS.

**TTS Providers:**

| Provider | Quality | Key | Network |
|----------|---------|-----|---------|
| **ElevenLabs** | ⭐⭐⭐⭐⭐ | `ELEVENLABS_API_KEY` | Required |
| **OpenAI** | ⭐⭐⭐⭐ | `OPENAI_API_KEY` | Required |
| **System (pyttsx3)** | ⭐⭐⭐ | Auto-detected | Offline |
| **Streaming TTS** (v5.1) | ⭐⭐⭐⭐ | Provider-dependent | Required |

### Voice Activity Detection (v5.1)

New in v5.1: VAD automatically detects when you start and stop speaking, eliminating the need for push-to-talk:

```yaml
voice:
  talk_mode:
    vad:
      enabled: true
      threshold: 0.5
      silence_duration_ms: 1500
```

### Streaming STT (v5.1)

New in v5.1: Streaming speech-to-text provides real-time transcription with lower latency:

```yaml
voice:
  talk_mode:
    streaming_stt:
      enabled: true
      chunk_duration_ms: 100
      language: "en-US"
```

```bash
# Start talk mode with streaming STT
manusclaw voice talk --start --streaming
```

---

## 10. Live Canvas (A2UI) v2

The Agent-to-UI protocol allows agents to render interactive UI components in real-time.

### Canvas v2 Features (New in v5.1)

- **Multi-user collaborative editing** — Multiple users can view and interact with the same canvas
- **Real-time sync** — Changes propagate instantly to all connected clients
- **Component versioning** — Track and revert component changes
- **New components:**

| Component | Type | Description |
|-----------|------|-------------|
| `TextComponent` | Text | Rich text with markdown |
| `ImageComponent` | Image | Base64 or URL images |
| `ButtonComponent` | Button | Clickable with callbacks |
| `TableComponent` | Table | Headers + rows |
| `ChartComponent` | Chart | Bar, line, pie, scatter, radar |
| `ContainerComponent` | Layout | Row, column, grid |
| `FormComponent` (v5.1) | Form | Input fields, dropdowns, checkboxes |
| `TabComponent` (v5.1) | Navigation | Tabbed interface |
| `AccordionComponent` (v5.1) | Collapsible | Expandable sections |

### Starting the Canvas

```bash
manusclaw-server
# Canvas WebSocket: ws://localhost:8765/ws/canvas/{session_id}
```

### Collaborative Canvas (v5.1)

```bash
# Start collaborative canvas session
manusclaw-server --canvas-mode collaborative

# Connect as a node
manusclaw-node --server ws://your-server:8765 --canvas-session shared-session-id
```

---

## 11. Security Configuration

### RBAC (Role-Based Access Control)

Define roles and permissions to control what users can do:

```yaml
security:
  rbac:
    enabled: true
    default_role: "user"
    roles:
      admin:
        permissions: ["*"]
      user:
        permissions:
          - "agent:execute"
          - "session:read"
          - "session:write"
          - "tools:use"
      viewer:
        permissions:
          - "session:read"
          - "agent:status"
```

### Input Validation

Protect against injection attacks and malformed inputs:

```yaml
security:
  input_validation:
    enabled: true
    strict_mode: false
    max_input_length: 10000
    sanitization:
      strip_html: true
      strip_control_chars: true
      reject_injection_patterns: true
    blocked_patterns:
      - "ignore previous instructions"
      - "system prompt"
```

### Rate Limiting

Prevent abuse with configurable rate limits:

```yaml
security:
  rate_limiting:
    enabled: true
    requests_per_minute: 60
    burst_allowance: 10
    per_user:
      enabled: true
      requests_per_minute: 20
```

### Audit Logging

Track all security-relevant events:

```yaml
security:
  audit_logging:
    enabled: true
    log_file: "~/.manusclaw/logs/audit.log"
    events:
      authentication: true
      authorization: true
      agent_execution: true
      tool_usage: true
      config_changes: true
```

---

## 12. Hooks System

Inject custom logic at specific points in the agent lifecycle.

### Hook Types

| Hook Type | When It Runs | Use Case |
|-----------|-------------|----------|
| `pre_execute` | Before agent processes a task | Input validation, logging, authorization |
| `post_execute` | After agent completes a task | Notification, archiving, metrics |
| `on_error` | When agent encounters an error | Alerting, retry logic, fallback |
| `on_session_start` | When a new session begins | Initialization, context loading |
| `on_session_end` | When a session ends | Cleanup, export, archival |
| `on_tool_call` | Before/after each tool call | Auditing, approval, rate limiting |

### Hook Implementations

**Shell hook:**

```yaml
hooks:
  pre_execute:
    - name: "log_request"
      type: "shell"
      command: "~/.manusclaw/hooks/pre_execute.sh"
      timeout: 30
      on_failure: "abort"  # abort | continue | warn
```

**Python hook:**

```yaml
hooks:
  pre_execute:
    - name: "validate_input"
      type: "python"
      module: "app.hooks.validation"
      function: "validate_agent_input"
      timeout: 10
      on_failure: "abort"
```

**Webhook hook:**

```yaml
hooks:
  post_execute:
    - name: "notify_completion"
      type: "webhook"
      url: "https://hooks.slack.com/services/xxx"
      method: "POST"
      timeout: 15
      on_failure: "warn"
```

---

## 13. Secrets Management

Securely store and retrieve sensitive configuration values.

### Supported Backends

| Backend | Description | Best For |
|---------|-------------|----------|
| `env` | Environment variables (default) | Development, simple deployments |
| `vault` | HashiCorp Vault | Production, enterprise |
| `encrypted_file` | Locally encrypted file | Single-server, no Vault |
| `aws_secrets_manager` | AWS Secrets Manager | AWS deployments |
| `gcp_secret_manager` | GCP Secret Manager | GCP deployments |

### HashiCorp Vault Configuration

```yaml
secrets:
  backend: "vault"
  vault:
    enabled: true
    address: "http://vault:8200"
    auth:
      method: "token"  # token | approle | kubernetes | ldap
    cache:
      enabled: true
      ttl_seconds: 300
    rotation:
      enabled: true
      interval_days: 30
```

### Secret Injection

Secrets can be automatically injected into environment variables:

```yaml
secrets:
  injection:
    enabled: true
    mappings:
      OPENAI_API_KEY: "openai/api_key"
      ANTHROPIC_API_KEY: "anthropic/api_key"
      DATABASE_URL: "database/url"
```

### CLI Commands

```bash
# Store a secret
manusclaw secrets set openai/api_key "sk-proj-xxx"

# Retrieve a secret
manusclaw secrets get openai/api_key

# List all secret paths
manusclaw secrets list

# Delete a secret
manusclaw secrets delete openai/api_key

# Rotate a secret
manusclaw secrets rotate openai/api_key
```

---

## 14. Context Management

Control how the agent's context window is used with compression, summarization, and optimization.

### Compression Strategies

| Strategy | Description | Best For |
|----------|-------------|----------|
| `sliding` | Keep recent messages, discard old ones | Simple conversations |
| `summarization` | Summarize old messages to save tokens | Long conversations |
| `hybrid` | Summarize old + keep recent | General use (recommended) |

```yaml
context:
  compression:
    enabled: true
    strategy: "hybrid"
    hybrid:
      summarize_older_than: 10
      keep_recent: 10
```

### Context Window Management

```yaml
context:
  window:
    max_tokens: 128000
    reserved_output: 4096
    reserved_system: 2048
```

### Long-Term Memory Integration

```yaml
context:
  memory:
    enabled: true
    auto_save: true
    save_trigger: "on_task_complete"
    max_memory_entries: 1000
```

---

## 15. Conversation Management

Manage threaded conversations with persistence and export capabilities.

### Threading

```yaml
conversation:
  threading:
    enabled: true
    max_depth: 10
    auto_title: true
```

### Persistence Backends

| Backend | Description | Best For |
|---------|-------------|----------|
| `file` | JSON files on disk | Single-server |
| `database` | SQLite/PostgreSQL | Production |
| `redis` | Redis key-value store | High-performance |

### Export

```bash
# Export a conversation
manusclaw conversations export --session abc123 --format markdown --output conversation.md

# List conversations
manusclaw conversations list

# Search conversations
manusclaw conversations search "deployment"
```

---

## 16. File Store

Pluggable storage backends for agent artifacts and file management.

### Supported Backends

| Backend | Description | Best For |
|---------|-------------|----------|
| `local` | Local filesystem | Development, single-server |
| `s3` | Amazon S3 | AWS deployments |
| `gcs` | Google Cloud Storage | GCP deployments |
| `azure_blob` | Azure Blob Storage | Azure deployments |

### S3 Configuration

```yaml
file_store:
  backend: "s3"
  s3:
    bucket: "manusclaw-artifacts"
    prefix: "manusclaw/"
    region: "us-east-1"
    encryption:
      enabled: true
      type: "AES256"
```

### CLI Commands

```bash
# Upload a file
manusclaw files upload report.pdf

# Download a file
manusclaw files download report.pdf --output ./downloads/

# List stored files
manusclaw files list

# Delete a file
manusclaw files delete report.pdf
```

---

## 17. Observability

Full OpenTelemetry integration for distributed tracing, metrics, and structured logging.

### OpenTelemetry Tracing

```yaml
observability:
  enabled: true
  telemetry:
    enabled: true
    service_name: "manusclaw"
    otlp:
      endpoint: "http://otel-collector:4317"
      protocol: "grpc"
    sampling:
      type: "trace_id_ratio"
      rate: 0.1
```

### Prometheus Metrics

```yaml
observability:
  metrics:
    enabled: true
    prometheus:
      enabled: true
      port: 9090
      path: "/metrics"
```

**Available metrics:**

- `manusclaw_agent_execution_time_seconds` — Agent execution duration
- `manusclaw_agent_execution_total` — Total agent executions
- `manusclaw_tool_call_total` — Tool invocations by type
- `manusclaw_tool_call_duration_seconds` — Tool call duration
- `manusclaw_llm_request_total` — LLM API requests
- `manusclaw_llm_token_usage_total` — Token usage by provider
- `manusclaw_channel_message_total` — Messages by channel
- `manusclaw_error_total` — Errors by type
- `manusclaw_parallel_tasks_active` — Active parallel tasks

### Structured Logging

```yaml
observability:
  logging:
    structured: true
    format: "json"
    include_trace_id: true
    include_span_id: true
```

---

## 18. Parallel Executor

Execute multiple agent tasks concurrently with worker pools and dependency graphs.

### Configuration

```yaml
parallel_executor:
  enabled: true
  mode: "threaded"  # threaded | process | ray
  workers:
    max_workers: 4
  dependency_graph:
    enabled: true
    cycle_detection: true
  retry:
    enabled: true
    max_retries: 3
    backoff_strategy: "exponential"
```

### Usage

```bash
# Execute tasks in parallel
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
```

---

## 19. Migration System

Handle database schema changes and configuration version upgrades automatically.

### Commands

```bash
# Check current migration status
manusclaw-migrate status

# Run pending migrations
manusclaw-migrate run

# Migrate from specific version
manusclaw-migrate --from 5.0.0 --to 5.1.0

# Rollback last migration
manusclaw-migrate rollback

# Create a new migration
manusclaw-migrate create --name "add_conversation_tables"
```

### Configuration

```yaml
migrations:
  auto_run: true
  backup:
    enabled: true
    max_backups: 10
  rollback:
    enabled: true
    auto_rollback_on_failure: false
```

---

## 20. Git Providers

Repository-aware agent operations with GitHub, GitLab, and Bitbucket integration.

### GitHub

```yaml
git_providers:
  github:
    enabled: true
    token: ""  # GITHUB_TOKEN env var
    permissions:
      read_repos: true
      write_repos: false
      read_issues: true
      write_issues: true
      read_pull_requests: true
```

### GitLab

```yaml
git_providers:
  gitlab:
    enabled: true
    token: ""  # GITLAB_TOKEN env var
    api_url: "https://gitlab.com/api/v4"
```

### Bitbucket

```yaml
git_providers:
  bitbucket:
    enabled: true
    username: ""
    app_password: ""
```

### Usage

Agents can use git provider tools to:

- List and browse repositories
- Read file contents from repos
- Create and manage issues
- Open and review pull requests
- Trigger CI/CD pipelines
- Search code across repositories

---

## 21. Integrations Plugin System

Connect ManusClaw to third-party services with a plugin architecture.

### Built-in Integrations

| Integration | Description | Key |
|-------------|-------------|-----|
| **Jira** | Issue tracking and project management | `JIRA_API_TOKEN` |
| **Notion** | Knowledge base and documentation | `NOTION_API_KEY` |
| **PagerDuty** | Incident management and alerting | `PAGERDUTY_API_KEY` |

### Custom Plugins

Create custom integration plugins by implementing the `IntegrationPlugin` interface:

```python
# ~/.manusclaw/plugins/my_plugin/__init__.py
from manusclaw.integrations import IntegrationPlugin

class MyPlugin(IntegrationPlugin):
    name = "my_plugin"
    version = "1.0.0"

    async def initialize(self, config):
        self.api_key = config.get("api_key")

    async def execute_action(self, action, params):
        if action == "send_notification":
            return await self._send_notification(params)
        raise ValueError(f"Unknown action: {action}")

    async def _send_notification(self, params):
        # Custom logic here
        return {"status": "sent"}
```

Register in config:

```yaml
integrations:
  plugins:
    enabled: true
    directory: "~/.manusclaw/plugins"
    configurations:
      my_plugin:
        enabled: true
        api_key: "your-key"
```

---

## 22. 12+ LLM Providers

### Supported Providers

| # | Provider | Config Key | Models (Examples) | Notes |
|---|----------|-----------|-------------------|-------|
| 1 | **OpenAI** | `openai` | gpt-4o, gpt-4o-mini, o1 | Most popular |
| 2 | **Anthropic** | `anthropic` | claude-sonnet-4-20250514, claude-opus | Long context, coding |
| 3 | **Google** | `google` | gemini-1.5-pro, gemini-1.5-flash | 1M token context |
| 4 | **Groq** | `groq` | llama-3.3-70b-versatile, mixtral | Ultra-fast inference |
| 5 | **Mistral** | `mistral` | mistral-large, codestral | European provider |
| 6 | **Ollama** | `ollama` | llama3, codellama, phi3 | Local models |
| 7 | **OpenRouter** | `openrouter` | Any from 100+ providers | Meta-provider |
| 8 | **HuggingFace** | `huggingface` | Any HF model | Free community models |
| 9 | **Together AI** | `together` | Llama, Mistral | Open-source models |
| 10 | **DeepInfra** | `deepinfra` | Llama, Mixtral | Competitive pricing |
| 11 | **Cohere** | `cohere` | command-r-plus | RAG-optimized |
| 12 | **Azure OpenAI** | `azure` | gpt-4o (Azure-hosted) | Enterprise Azure |

### Switching Providers

```bash
# Via CLI flags
manusclaw --provider groq --model llama-3.3-70b-versatile

# Via environment variables
export MANUSCLAW_PROVIDER=anthropic
export MANUSCLAW_MODEL=claude-sonnet-4-20250514

# Via config.toml
[llm]
provider = "groq"
model = "llama-3.3-70b-versatile"
```

---

## 23. Enhanced Cron

Schedule recurring agent tasks with channel output delivery, dependencies, and retry policies.

### v5.1 Enhancements

- **Cron job dependencies** — Jobs can depend on other jobs completing first
- **Retry policies** — Automatic retries with exponential backoff
- **Distributed scheduling** — Prevent duplicate execution in multi-instance deployments

### Configuration

```yaml
# ~/.manusclaw/cron.yaml
jobs:
  - name: "Daily Report"
    cron: "0 9 * * *"
    prompt: "Generate the daily summary report"
    output:
      channel: telegram
      chat_id: "-1001234567890"
    retry:
      enabled: true
      max_retries: 3
      backoff_seconds: 60

  - name: "Weekly Analysis"
    cron: "0 9 * * 1"
    prompt: "Run the weekly data analysis"
    depends_on: ["Daily Report"]  # v5.1: wait for daily report first
    retry:
      enabled: true
      max_retries: 2

  - name: "Health Check"
    cron: "*/15 * * * *"
    prompt: "Check system health and report any issues"
    retry:
      enabled: true
      max_retries: 5
      backoff_seconds: 30
```

---

## 24. Session Tools CLI

```bash
# List all sessions
manusclaw-sessions list

# Show session history with tool calls
manusclaw-sessions history --session abc123 --tool-calls

# Send a message to a running session
manusclaw-sessions send --session abc123 --message "Continue the task"

# Spawn a new session + agent run
manusclaw-sessions spawn --prompt "Analyze the data"

# Delete a session
manusclaw-sessions delete --session abc123 --force

# Export session as JSON
manusclaw-sessions export --session abc123 --output session.json
```

---

## 25. Sandbox Backends

Three sandbox backends for code isolation:

| Backend | Use Case | Config |
|---------|----------|--------|
| **Docker** | Container isolation (network=none) | `SANDBOX_BACKEND=docker` |
| **SSH** | Remote execution on dedicated host | `SANDBOX_BACKEND=ssh` |
| **OpenShell** | Lightweight Linux namespace isolation | `SANDBOX_BACKEND=openshell` |

```toml
[sandbox]
enabled = true
backend = "docker"
docker_image = "python:3.12-slim"
memory_limit = "2g"
timeout = 300
```

---

## 26. Companion Apps

### Desktop GUI (Flet)

```bash
pip install flet
manusclaw-desktop
```

### macOS Menu Bar

```bash
pip install rumps websockets
manusclaw-menubar
```

### Windows System Tray

```bash
pip install pystray Pillow websockets
manusclaw-hub
```

### Mobile Node Client

```bash
pip install websockets
manusclaw-node --server ws://your-server:8765
```
