# Configuration Guide — ManusClaw v5.1.0

ManusClaw v5.1.0 is configured through a layered system of configuration files, environment variables, and profiles. The three primary mechanisms are:

- **`config.yaml`** (v5.0+) — Structured settings for model profiles, agent definitions and routing, voice, SSH server, channel integrations, security, hooks, context, conversation, observability, secrets, file store, git providers, integrations, parallel executor, and migrations. This is the canonical configuration format.
- **`config.toml`** — Legacy structured settings for LLM providers, search, token budgets, permissions, workspace, memory, logging, and server. Fully supported but superseded by `config.yaml` for v5.1 features.
- **`.env`** — Sensitive credentials like API keys, tokens, and secrets. Never committed to version control.

Understanding how these files interact through the **8-layer config priority chain** and the **config profiles system** is essential for getting the most out of ManusClaw. This guide explains every configuration option in detail, with examples for each LLM provider and use case.

---

## Table of Contents

- [Configuration File Locations](#configuration-file-locations)
- [8-Layer Config Priority Chain](#8-layer-config-priority-chain)
- [Config Profiles](#config-profiles)
- [config.yaml Reference](#configyaml-reference)
  - [Model Profiles](#model-profiles)
  - [Agent Definitions](#agent-definitions)
  - [Agent Routes](#agent-routes)
  - [Voice Configuration](#voice-configuration)
  - [SSH Server Configuration](#ssh-server-configuration)
  - [Channels Configuration](#channels-configuration)
  - [Security Configuration (v5.1)](#security-configuration-v51)
  - [Hooks System (v5.1)](#hooks-system-v51)
  - [Context Management (v5.1)](#context-management-v51)
  - [Conversation Management (v5.1)](#conversation-management-v51)
  - [Observability Configuration (v5.1)](#observability-configuration-v51)
  - [Secrets Management (v5.1)](#secrets-management-v51)
  - [File Store Configuration (v5.1)](#file-store-configuration-v51)
  - [Git Providers (v5.1)](#git-providers-v51)
  - [Integrations (v5.1)](#integrations-v51)
  - [Parallel Executor (v5.1)](#parallel-executor-v51)
  - [Migrations Configuration (v5.1)](#migrations-configuration-v51)
  - [Full Example config.yaml](#full-example-configyaml)
- [config.toml Reference](#configtoml-reference)
  - [LLM Configuration](#llm-configuration)
  - [Provider-Specific Settings](#provider-specific-settings)
  - [Search Engine Configuration](#search-engine-configuration)
  - [Token Budget Configuration](#token-budget-configuration)
  - [Permissions Configuration](#permissions-configuration)
  - [Workspace Configuration](#workspace-configuration)
  - [Memory Configuration](#memory-configuration)
  - [Logging Configuration](#logging-configuration)
  - [Server Configuration](#server-configuration)
  - [Sandbox Configuration](#sandbox-configuration)
- [Environment Variables Reference](#environment-variables-reference)
- [Migrating from v5.0.0 to v5.1.0](#migrating-from-v500-to-v510)

---

## Configuration File Locations

ManusClaw looks for configuration files in the following locations, in order:

| Priority | Location | Notes |
|----------|----------|-------|
| 1 | `MANUSCLAW_CONFIG_DIR` env var | Custom config directory |
| 2 | `~/.manusclaw/` | Default user config directory |
| 3 | `/etc/manusclaw/` | System-wide config (Linux only) |
| 4 | `./` | Current working directory |

The directory structure:

```
~/.manusclaw/
├── config.yaml           Primary configuration (v5.0+)
├── config.toml           Legacy configuration (still supported)
├── .env                  API keys and secrets
├── cron.yaml             Cron job definitions
├── profiles/             Config profiles
│   ├── default.yaml
│   ├── production.yaml
│   └── development.yaml
├── hooks/                Hook scripts (v5.1)
│   ├── pre_execute.sh
│   ├── post_execute.sh
│   └── on_error.sh
├── secrets/              Encrypted secrets (v5.1)
│   └── vault-cache/
├── migrations/           Migration state (v5.1)
│   └── .migration-state
├── context/              Context management state (v5.1)
│   └── summaries/
├── conversations/        Conversation persistence (v5.1)
├── sessions/             Session data
├── skills/               Custom skill definitions
├── nodes/                Canvas node state
├── ssh/                  SSH gateway keys
└── logs/                 Log files
```

---

## 8-Layer Config Priority Chain

ManusClaw v5.1.0 uses an 8-layer priority chain (expanded from 7 in v5.0.0). Higher-priority layers override lower ones:

| Layer | Source | Example | Priority |
|-------|--------|---------|----------|
| 1 | CLI flags | `--provider groq --model llama3` | Highest |
| 2 | Environment variables | `MANUSCLAW_PROVIDER=groq` | |
| 3 | Runtime overrides | `manusclaw config set llm.provider groq` | |
| 4 | Active config profile | `profiles/production.yaml` | |
| 5 | `config.yaml` | Main config file | |
| 6 | `config.toml` | Legacy config file | |
| 7 | `.env` file | Environment file | |
| 8 | Built-in defaults | Hardcoded in source | Lowest |

When the same setting is defined at multiple layers, the highest-priority value wins. For example, `--provider groq` on the CLI overrides `provider: openai` in `config.yaml`, which overrides `provider = "anthropic"` in `config.toml`.

### New in v5.1.0: Runtime Overrides (Layer 3)

Layer 3 is new in v5.1.0. Runtime overrides allow you to change configuration values without editing files:

```bash
# Set a runtime override
manusclaw config set llm.provider groq

# List all runtime overrides
manusclaw config list --runtime

# Clear a runtime override
manusclaw config unset llm.provider

# Clear all runtime overrides
manusclaw config reset
```

Runtime overrides persist across sessions in `~/.manusclaw/runtime.yaml`.

---

## Config Profiles

Config profiles allow you to maintain multiple configurations and switch between them easily. This is especially useful for:

- **Development vs. production** — Different LLM providers, token budgets, and permission modes
- **Cost optimization** — Cheap models for testing, powerful models for production
- **Team sharing** — Each team member can have their own profile
- **Feature flags** — Enable/disable v5.1 features per profile

### Creating a Profile

```bash
# Create a new profile from the current config
manusclaw profile create production

# Create a profile from a template
manusclaw profile create development --template minimal

# Switch to a profile
manusclaw profile switch production

# List all profiles
manusclaw profile list
```

### Profile File Structure

Each profile is a YAML file in `~/.manusclaw/profiles/`:

```yaml
# ~/.manusclaw/profiles/production.yaml
name: production
description: "Production configuration with Claude and strict security"
inherits: default  # Inherit from another profile

overrides:
  model_profiles:
    default:
      - provider: anthropic
        model: claude-sonnet-4-20250514
        priority: 1
  security:
    rbac:
      enabled: true
    input_validation:
      strict_mode: true
    audit_logging:
      enabled: true
  observability:
    enabled: true
    tracing:
      sampling_rate: 1.0  # 100% in production
  parallel_executor:
    max_workers: 8
```

---

## config.yaml Reference

### Model Profiles

Model profiles define failover chains for LLM providers. When the primary provider fails, ManusClaw automatically switches to the next provider in the chain.

```yaml
model_profiles:
  # Default profile — used when no profile is specified
  default:
    - provider: groq
      model: llama-3.3-70b-versatile
      priority: 1
      max_tokens: 4096
      temperature: 0.7
    - provider: openai
      model: gpt-4o
      priority: 2
    - provider: anthropic
      model: claude-sonnet-4-20250514
      priority: 3

  # Fast profile — optimized for speed
  fast:
    - provider: groq
      model: llama-3.3-70b-versatile
      priority: 1
    - provider: openai
      model: gpt-4o-mini
      priority: 2

  # Quality profile — optimized for best results
  quality:
    - provider: anthropic
      model: claude-sonnet-4-20250514
      priority: 1
    - provider: openai
      model: gpt-4o
      priority: 2

  # Local profile — no API calls
  local:
    - provider: ollama
      model: llama3
      priority: 1
```

**Using a profile:**

```bash
manusclaw --profile fast "Quick question"
manusclaw --profile quality "Complex analysis"
```

### Agent Definitions

Define custom agent types with their Python class paths:

```yaml
agents:
  definitions:
    - name: manus
      class_path: app.agent.manus.Manus
      description: "Default Manus agent with full tool access"

    - name: data_analyst
      class_path: app.agent.data_analysis.DataAnalysisAgent
      description: "Specialized for data analysis tasks"

    - name: browser_agent
      class_path: app.agent.browser.BrowserAgent
      description: "Specialized for web browsing and scraping"

    - name: code_reviewer
      class_path: app.agent.code_review.CodeReviewAgent
      description: "Specialized for code review (v5.1)"

    - name: devops
      class_path: app.agent.devops.DevOpsAgent
      description: "Specialized for DevOps and infrastructure tasks (v5.1)"
```

### Agent Routes

Route incoming messages to specific agent types based on channel, user, or content patterns:

```yaml
agents:
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

    - pattern: "user_id:admin_user"
      agent: manus
      priority: 4

    - pattern: "keyword:deploy,kubernetes,docker"
      agent: devops
      priority: 5
```

### Voice Configuration

```yaml
voice:
  wake_word:
    enabled: true
    word: "hey manus"
    sensitivity: 0.7
    backend: auto  # auto | porcupine | google | streaming (v5.1)

  talk_mode:
    enabled: true
    stt_engine: auto  # auto | google | whisper | streaming (v5.1)
    tts_provider: auto  # auto | elevenlabs | openai | system | streaming (v5.1)

    # New in v5.1: Voice activity detection
    vad:
      enabled: true
      threshold: 0.5
      silence_duration_ms: 1500

    # New in v5.1: Streaming STT
    streaming_stt:
      enabled: true
      chunk_duration_ms: 100
      language: "en-US"

  stop_phrases:
    - "stop listening"
    - "go to sleep"
    - "goodbye"
```

### SSH Server Configuration

```yaml
ssh:
  enabled: false
  port: 2222
  auth_keys: ~/.ssh/authorized_keys
  host_key: ~/.manusclaw/ssh/host_key
  max_sessions: 5
  idle_timeout: 300  # seconds
  command_timeout: 120  # seconds
  allowed_commands:
    - status
    - restart
    - logs
    - agent
    - channels
    - cron
    - help
    - exit
    - config  # New in v5.1
```

### Channels Configuration

```yaml
channels:
  telegram:
    enabled: false
    bot_token: ""  # TELEGRAM_BOT_TOKEN env var

  discord:
    enabled: false
    bot_token: ""  # DISCORD_BOT_TOKEN env var

  slack:
    enabled: false
    bot_token: ""  # SLACK_BOT_TOKEN env var

  whatsapp:
    enabled: false
    access_token: ""  # WHATSAPP_ACCESS_TOKEN env var
    phone_id: ""      # WHATSAPP_BUSINESS_PHONE_ID env var

  signal:
    enabled: false
    rest_url: "http://localhost:8080"
    phone_number: ""  # SIGNAL_CLI_NUMBER env var

  matrix:
    enabled: false
    homeserver: "https://matrix.org"
    access_token: ""  # MATRIX_ACCESS_TOKEN env var
    user_id: ""       # MATRIX_USER_ID env var

  irc:
    enabled: false
    server: "irc.libera.chat"
    port: 6697
    nick: "manusclaw-bot"
    channels: []

  twitch:
    enabled: false
    bot_token: ""  # TWITCH_BOT_TOKEN env var
    channel: ""    # TWITCH_CHANNEL env var

  webchat:
    enabled: true  # Always available

  teams:
    enabled: false
    app_id: ""       # MICROSOFT_APP_ID env var
    app_password: "" # MICROSOFT_APP_PASSWORD env var

  google_chat:
    enabled: false
    service_account: ""  # GOOGLE_CHAT_SERVICE_ACCOUNT env var

  line:
    enabled: false  # New in v5.1
    channel_secret: ""  # LINE_CHANNEL_SECRET env var
    channel_access_token: ""  # LINE_CHANNEL_ACCESS_TOKEN env var

  email:
    enabled: false
    smtp_host: ""
    smtp_port: 587
    user: ""
    password: ""
```

### Security Configuration (v5.1)

The security configuration controls access policies, input validation, rate limiting, and audit logging.

```yaml
security:
  # Role-Based Access Control
  rbac:
    enabled: false
    default_role: "user"  # Default role for unauthenticated access
    roles:
      admin:
        permissions: ["*"]  # All permissions
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

    # User-to-role mappings
    users:
      admin_user:
        role: admin
        channels: ["telegram", "discord"]

  # Input validation and sanitization
  input_validation:
    enabled: true
    strict_mode: false  # In strict mode, reject any input that doesn't pass all validators
    max_input_length: 10000  # Maximum character length for user inputs
    max_prompt_length: 50000  # Maximum character length for agent prompts

    # Sanitization rules
    sanitization:
      strip_html: true          # Remove HTML tags from inputs
      strip_control_chars: true  # Remove control characters
      normalize_whitespace: true # Normalize whitespace
      reject_injection_patterns: true  # Reject common injection patterns

    # Blocked patterns (regex)
    blocked_patterns:
      - "ignore previous instructions"
      - "system prompt"
      - "jailbreak"

  # Rate limiting
  rate_limiting:
    enabled: true
    requests_per_minute: 60        # Global rate limit
    requests_per_hour: 1000        # Hourly rate limit
    burst_allowance: 10            # Allow brief bursts above the limit

    # Per-channel rate limits
    channels:
      telegram:
        requests_per_minute: 30
      discord:
        requests_per_minute: 30
      slack:
        requests_per_minute: 30
      webchat:
        requests_per_minute: 120

    # Per-user rate limits
    per_user:
      enabled: true
      requests_per_minute: 20
      requests_per_hour: 500

  # Sandbox restrictions
  sandbox:
    restrict_file_access: true     # Restrict file operations to workspace
    restrict_network_access: false  # Restrict network access from sandbox
    allowed_paths: []              # Additional allowed paths beyond workspace
    blocked_commands:              # Commands that agents cannot execute
      - "rm -rf /"
      - "mkfs"
      - "dd if=/dev/zero"
    max_execution_time: 300        # Maximum execution time in seconds

  # Audit logging
  audit_logging:
    enabled: false
    log_file: "~/.manusclaw/logs/audit.log"
    log_level: "INFO"  # DEBUG | INFO | WARNING | ERROR
    include_request_body: false    # Log full request bodies (security sensitive)
    include_response_body: false   # Log full response bodies (security sensitive)
    retention_days: 90             # How long to keep audit logs
    format: "json"                 # json | text

    # Events to log
    events:
      authentication: true         # Login/logout events
      authorization: true          # Permission check failures
      agent_execution: true        # Agent task executions
      tool_usage: true             # Tool invocations
      config_changes: true         # Configuration modifications
      data_access: false           # File/data access (verbose)
```

### Hooks System (v5.1)

Hooks allow you to inject custom logic at specific points in the agent lifecycle. Hooks can be shell scripts, Python functions, or HTTP webhooks.

```yaml
hooks:
  # Pre-execution hooks run before an agent processes a task
  pre_execute:
    - name: "log_request"
      type: "shell"  # shell | python | webhook
      command: "~/.manusclaw/hooks/pre_execute.sh"
      timeout: 30
      on_failure: "abort"  # abort | continue | warn

    - name: "validate_input"
      type: "python"
      module: "app.hooks.validation"
      function: "validate_agent_input"
      timeout: 10
      on_failure: "abort"

  # Post-execution hooks run after an agent completes a task
  post_execute:
    - name: "notify_completion"
      type: "webhook"
      url: "https://hooks.slack.com/services/xxx"
      method: "POST"
      headers:
        Content-Type: "application/json"
      timeout: 15
      on_failure: "warn"

    - name: "archive_result"
      type: "shell"
      command: "~/.manusclaw/hooks/post_execute.sh"
      timeout: 60
      on_failure: "warn"

  # Error hooks run when an agent encounters an error
  on_error:
    - name: "alert_error"
      type: "webhook"
      url: "https://hooks.slack.com/services/yyy"
      method: "POST"
      timeout: 10
      on_failure: "continue"

  # Session lifecycle hooks
  on_session_start:
    - name: "session_init"
      type: "python"
      module: "app.hooks.sessions"
      function: "on_session_start"
      timeout: 5
      on_failure: "warn"

  on_session_end:
    - name: "session_cleanup"
      type: "python"
      module: "app.hooks.sessions"
      function: "on_session_end"
      timeout: 10
      on_failure: "warn"

  # Tool usage hooks
  on_tool_call:
    - name: "audit_tool"
      type: "python"
      module: "app.hooks.audit"
      function: "audit_tool_usage"
      timeout: 5
      on_failure: "warn"
```

**Shell hook example** (`~/.manusclaw/hooks/pre_execute.sh`):

```bash
#!/bin/bash
# Pre-execution hook
# Environment variables available:
#   MANUSCLAW_SESSION_ID - Current session ID
#   MANUSCLAW_TASK_ID - Current task ID
#   MANUSCLAW_PROMPT - The user's prompt
#   MANUSCLAW_AGENT - The agent name

echo "[$(date -Iseconds)] Agent ${MANUSCLAW_AGENT} executing task ${MANUSCLAW_TASK_ID}" \
  >> ~/.manusclaw/logs/hook.log

# Return 0 to continue, non-zero to abort
exit 0
```

**Python hook example**:

```python
# app/hooks/validation.py
from typing import Dict, Any

async def validate_agent_input(context: Dict[str, Any]) -> bool:
    """Validate agent input before execution."""
    prompt = context.get("prompt", "")

    # Check for blocked patterns
    blocked = ["ignore previous", "system prompt"]
    for pattern in blocked:
        if pattern.lower() in prompt.lower():
            return False  # Abort execution

    return True  # Allow execution
```

### Context Management (v5.1)

Context management controls how the agent's context window is used, including compression, summarization, and optimization strategies.

```yaml
context:
  # Context window configuration
  window:
    max_tokens: 128000        # Maximum context window size
    reserved_output: 4096     # Tokens reserved for the response
    reserved_system: 2048     # Tokens reserved for system prompts

  # Compression strategy
  compression:
    enabled: true
    strategy: "sliding"  # sliding | summarization | hybrid

    # Sliding window strategy
    sliding:
      keep_recent_messages: 20  # Always keep the last N messages
      keep_system_messages: true
      keep_tool_results: 5      # Keep last N tool results

    # Summarization strategy
    summarization:
      model: null  # Use the same model as the agent (null) or specify a cheaper one
      max_summary_tokens: 2000
      trigger_threshold: 0.8  # Summarize when context is 80% full
      summary_prompt: "Summarize the conversation so far, preserving key facts, decisions, and tool outputs."

    # Hybrid strategy (summarize old + keep recent)
    hybrid:
      summarize_older_than: 10  # Summarize messages older than N
      keep_recent: 10           # Always keep the last N messages uncompressed

  # Context persistence
  persistence:
    enabled: true
    backend: "file"  # file | database | redis
    path: "~/.manusclaw/context"

  # Long-term memory integration
  memory:
    enabled: true
    auto_save: true
    save_trigger: "on_task_complete"  # on_task_complete | on_session_end | periodic
    max_memory_entries: 1000
```

### Conversation Management (v5.1)

Conversation management handles threaded conversations, persistence, and export.

```yaml
conversation:
  # Threading configuration
  threading:
    enabled: true
    max_depth: 10            # Maximum nesting depth for conversation threads
    max_threads_per_user: 50 # Maximum active threads per user
    auto_title: true         # Auto-generate thread titles
    title_model: null        # Model for title generation (null = default)

  # Persistence configuration
  persistence:
    enabled: true
    backend: "file"  # file | database | redis
    path: "~/.manusclaw/conversations"

    # Auto-save configuration
    auto_save:
      enabled: true
      interval_seconds: 60   # Save every 60 seconds
      on_message: true       # Save after each message

  # Export configuration
  export:
    formats: ["json", "markdown", "html"]
    include_metadata: true
    include_tool_calls: true
    max_export_size_mb: 50

  # Conversation retention
  retention:
    max_age_days: 365        # Delete conversations older than this
    max_conversations: 1000  # Maximum stored conversations
    auto_cleanup: false      # Automatically delete old conversations
```

### Observability Configuration (v5.1)

Observability provides OpenTelemetry integration, distributed tracing, metrics, and structured logging.

```yaml
observability:
  enabled: false

  # OpenTelemetry configuration
  telemetry:
    enabled: false
    service_name: "manusclaw"
    service_version: "5.1.0"

    # OTLP exporter configuration
    otlp:
      endpoint: "http://localhost:4317"  # OTLP gRPC endpoint
      protocol: "grpc"  # grpc | http
      headers: {}       # Additional headers
      timeout: 30       # Timeout in seconds

    # Sampling configuration
    sampling:
      type: "trace_id_ratio"  # always_on | always_off | trace_id_ratio
      rate: 0.1  # Sample 10% of traces (increase in production)

  # Distributed tracing
  tracing:
    enabled: false
    trace_agent_executions: true
    trace_tool_calls: true
    trace_llm_requests: true
    trace_channel_messages: true
    trace_hooks: false  # Whether to trace hook executions
    max_trace_attributes: 128

  # Metrics
  metrics:
    enabled: false
    port: 9090  # Prometheus metrics endpoint

    # Metrics to collect
    collect:
      agent_execution_time: true
      agent_execution_count: true
      tool_call_count: true
      tool_call_duration: true
      llm_request_count: true
      llm_token_usage: true
      llm_request_duration: true
      channel_message_count: true
      session_count: true
      error_count: true
      hook_execution_time: true
      parallel_task_count: true

    # Prometheus configuration
    prometheus:
      enabled: false
      port: 9090
      path: "/metrics"

  # Structured logging
  logging:
    structured: false  # Use structured JSON logging
    level: "INFO"      # DEBUG | INFO | WARNING | ERROR | CRITICAL
    format: "text"     # text | json
    include_trace_id: true
    include_span_id: true
    output:
      - type: "console"
      - type: "file"
        path: "~/.manusclaw/logs/manusclaw.jsonl"
        rotation: "daily"
        max_size_mb: 100
        retention_days: 30
```

### Secrets Management (v5.1)

Secrets management provides secure storage and retrieval of sensitive configuration values, with support for HashiCorp Vault and encrypted local storage.

```yaml
secrets:
  # Backend configuration
  backend: "env"  # env | vault | encrypted_file | aws_secrets_manager | gcp_secret_manager

  # HashiCorp Vault configuration
  vault:
    enabled: false
    address: "http://localhost:8200"
    token: ""  # VAULT_TOKEN env var
    namespace: ""
    mount_point: "secret"
    path: "manusclaw"

    # Authentication methods
    auth:
      method: "token"  # token | approle | kubernetes | ldap
      # For AppRole:
      role_id: ""
      secret_id: ""
      # For Kubernetes:
      kubernetes_role: ""
      kubernetes_jwt_path: "/var/run/secrets/kubernetes.io/serviceaccount/token"

    # TLS configuration
    tls:
      verify: true
      ca_cert: ""
      client_cert: ""
      client_key: ""

    # Cache configuration
    cache:
      enabled: true
      ttl_seconds: 300  # Cache secrets for 5 minutes
      max_entries: 256

    # Secret rotation
    rotation:
      enabled: false
      interval_days: 30
      auto_rotate: false  # Automatically rotate secrets

  # Encrypted file storage (for non-Vault environments)
  encrypted_file:
    enabled: false
    path: "~/.manusclaw/secrets/secrets.enc"
    key_derivation: "pbkdf2"  # pbkdf2 | argon2
    key_file: "~/.manusclaw/secrets/.key"

  # AWS Secrets Manager (alternative to Vault)
  aws_secrets_manager:
    enabled: false
    region: "us-east-1"
    secret_name: "manusclaw/secrets"
    access_key_id: ""    # AWS_ACCESS_KEY_ID env var
    secret_access_key: "" # AWS_SECRET_ACCESS_KEY env var

  # GCP Secret Manager (alternative to Vault)
  gcp_secret_manager:
    enabled: false
    project_id: ""
    secret_name: "manusclaw-secrets"
    credentials_file: ""  # GOOGLE_APPLICATION_CREDENTIALS env var

  # Secret injection
  injection:
    # Automatically inject secrets into environment variables
    enabled: true
    # Mapping of env var name to secret path
    mappings:
      OPENAI_API_KEY: "openai/api_key"
      ANTHROPIC_API_KEY: "anthropic/api_key"
      DATABASE_URL: "database/url"
```

### File Store Configuration (v5.1)

The file store provides pluggable storage backends for agent artifacts, uploaded files, and generated outputs.

```yaml
file_store:
  # Backend configuration
  backend: "local"  # local | s3 | gcs | azure_blob

  # Local filesystem storage
  local:
    base_path: "~/.manusclaw/files"
    max_file_size_mb: 100
    allowed_extensions: []  # Empty = all allowed; e.g., [".txt", ".py", ".json"]
    create_subdirs: true    # Create date-based subdirectories (YYYY/MM/DD)

  # Amazon S3 storage
  s3:
    bucket: ""
    prefix: "manusclaw/"
    region: "us-east-1"
    access_key_id: ""     # AWS_ACCESS_KEY_ID env var
    secret_access_key: "" # AWS_SECRET_ACCESS_KEY env var
    endpoint_url: ""      # For S3-compatible services (MinIO, etc.)
    storage_class: "STANDARD"  # STANDARD | STANDARD_IA | GLACIER
    encryption:
      enabled: false
      type: "AES256"  # AES256 | aws:kms
      kms_key_id: ""

  # Google Cloud Storage
  gcs:
    bucket: ""
    prefix: "manusclaw/"
    project_id: ""
    credentials_file: ""  # GOOGLE_APPLICATION_CREDENTIALS env var
    encryption:
      enabled: false
      kms_key_name: ""

  # Azure Blob Storage
  azure_blob:
    container: ""
    prefix: "manusclaw/"
    connection_string: ""  # AZURE_STORAGE_CONNECTION_STRING env var
    account_name: ""
    account_key: ""
    encryption:
      enabled: false

  # Common settings
  cleanup:
    enabled: false
    max_age_days: 30
    max_total_size_mb: 5000
    schedule: "0 2 * * *"  # Cron schedule for cleanup
```

### Git Providers (v5.1)

Git provider integration enables repository-aware agent operations, including reading files, creating issues, opening pull requests, and managing repositories.

```yaml
git_providers:
  # GitHub integration
  github:
    enabled: false
    token: ""  # GITHUB_TOKEN env var
    default_owner: ""
    default_repo: ""
    api_url: "https://api.github.com"
    rate_limit:
      requests_per_hour: 5000  # GitHub API rate limit
    permissions:
      read_repos: true
      write_repos: false
      read_issues: true
      write_issues: true
      read_pull_requests: true
      write_pull_requests: false
      read_actions: true
      write_actions: false

  # GitLab integration
  gitlab:
    enabled: false
    token: ""  # GITLAB_TOKEN env var
    default_group: ""
    default_project: ""
    api_url: "https://gitlab.com/api/v4"  # Change for self-hosted
    permissions:
      read_projects: true
      write_projects: false
      read_merge_requests: true
      write_merge_requests: true
      read_pipelines: true
      write_pipelines: false

  # Bitbucket integration
  bitbucket:
    enabled: false
    username: ""  # BITBUCKET_USERNAME env var
    app_password: ""  # BITBUCKET_APP_PASSWORD env var
    default_workspace: ""
    default_repo: ""
    api_url: "https://api.bitbucket.org/2.0"
    permissions:
      read_repos: true
      write_repos: false
      read_pull_requests: true
      write_pull_requests: true
      read_pipelines: true
```

### Integrations (v5.1)

The integrations system provides a plugin architecture for connecting ManusClaw to third-party services.

```yaml
integrations:
  # Jira integration
  jira:
    enabled: false
    server_url: ""
    username: ""
    api_token: ""  # JIRA_API_TOKEN env var
    default_project: ""
    auto_create_issues: false
    auto_update_issues: true
    custom_fields: {}

  # Notion integration
  notion:
    enabled: false
    api_key: ""  # NOTION_API_KEY env var
    default_database: ""
    auto_create_pages: false
    page_template: ""

  # PagerDuty integration
  pagerduty:
    enabled: false
    api_key: ""  # PAGERDUTY_API_KEY env var
    routing_key: ""
    severity_map:
      critical: "critical"
      error: "error"
      warning: "warning"

  # Custom integration plugin system
  plugins:
    enabled: true
    directory: "~/.manusclaw/plugins"

    # Plugin configuration
    # Each plugin is a Python module that implements the IntegrationPlugin interface
    configurations: {}
    # Example:
    # my_custom_plugin:
    #   enabled: true
    #   setting1: "value1"
    #   setting2: 42
```

### Parallel Executor (v5.1)

The parallel executor enables concurrent agent execution with worker pools, dependency graphs, and resource management.

```yaml
parallel_executor:
  enabled: false

  # Execution mode
  mode: "threaded"  # threaded | process | ray

  # Worker pool configuration
  workers:
    max_workers: 4          # Maximum concurrent workers
    min_workers: 1          # Minimum workers to keep alive
    idle_timeout: 300       # Seconds before idle workers are stopped
    startup_timeout: 60     # Seconds to wait for worker startup

  # Task queue configuration
  queue:
    max_size: 100           # Maximum queued tasks
    priority_enabled: true  # Enable priority-based scheduling
    default_priority: 5     # Default task priority (1=highest, 10=lowest)

  # Dependency graph configuration
  dependency_graph:
    enabled: true           # Enable task dependencies
    max_depth: 10           # Maximum dependency chain depth
    cycle_detection: true   # Detect and prevent circular dependencies

  # Resource limits
  resources:
    max_memory_per_task_mb: 512  # Maximum memory per task
    max_cpu_per_task: 1.0        # Maximum CPU cores per task
    max_time_per_task_seconds: 600  # Maximum execution time per task

  # Retry configuration
  retry:
    enabled: true
    max_retries: 3
    backoff_strategy: "exponential"  # fixed | linear | exponential
    initial_delay_seconds: 5
    max_delay_seconds: 300

  # Ray configuration (for distributed execution)
  ray:
    address: "auto"  # auto-detect | ray://cluster-address:10001
    num_cpus: 4
    num_gpus: 0
    object_store_memory: 1000000000  # 1GB
```

### Migrations Configuration (v5.1)

The migration system handles database schema changes and configuration version upgrades.

```yaml
migrations:
  # Migration directory
  directory: "~/.manusclaw/migrations"

  # Auto-run migrations on startup
  auto_run: true

  # Migration version tracking
  current_version: "5.1.0"

  # Backup before migration
  backup:
    enabled: true
    directory: "~/.manusclaw/migrations/backups"
    max_backups: 10
    include_database: true
    include_config: true
    include_sessions: true

  # Migration hooks
  pre_migration:
    - type: "shell"
      command: "~/.manusclaw/hooks/pre_migration.sh"
      timeout: 60

  post_migration:
    - type: "shell"
      command: "~/.manusclaw/hooks/post_migration.sh"
      timeout: 60

  # Rollback configuration
  rollback:
    enabled: true
    auto_rollback_on_failure: false  # Automatically rollback if migration fails
    max_rollback_depth: 5
```

### Full Example config.yaml

```yaml
# ~/.manusclaw/config.yaml — ManusClaw v5.1.0 Full Configuration

# ── Model Profiles ──────────────────────────────────────────────
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

# ── Agents ──────────────────────────────────────────────────────
agents:
  definitions:
    - name: manus
      class_path: app.agent.manus.Manus
    - name: data_analyst
      class_path: app.agent.data_analysis.DataAnalysisAgent

  routes:
    - pattern: "channel:telegram"
      agent: manus
      priority: 1
    - pattern: "channel:discord,#analytics"
      agent: data_analyst
      priority: 2

# ── Voice ───────────────────────────────────────────────────────
voice:
  wake_word:
    enabled: true
    word: "hey manus"
    backend: auto
  talk_mode:
    enabled: true
    stt_engine: auto
    tts_provider: auto
    vad:
      enabled: true
    streaming_stt:
      enabled: true

# ── SSH ─────────────────────────────────────────────────────────
ssh:
  enabled: false
  port: 2222
  auth_keys: ~/.ssh/authorized_keys

# ── Channels ────────────────────────────────────────────────────
channels:
  telegram:
    enabled: true
  webchat:
    enabled: true

# ── Security (v5.1) ────────────────────────────────────────────
security:
  rbac:
    enabled: false
    default_role: "user"
  input_validation:
    enabled: true
    strict_mode: false
    max_input_length: 10000
  rate_limiting:
    enabled: true
    requests_per_minute: 60
  sandbox:
    restrict_file_access: true
    restrict_network_access: false
  audit_logging:
    enabled: false

# ── Hooks (v5.1) ───────────────────────────────────────────────
hooks:
  pre_execute: []
  post_execute: []
  on_error: []

# ── Context Management (v5.1) ──────────────────────────────────
context:
  window:
    max_tokens: 128000
    reserved_output: 4096
  compression:
    enabled: true
    strategy: "sliding"
  persistence:
    enabled: true
    backend: "file"

# ── Conversation Management (v5.1) ─────────────────────────────
conversation:
  threading:
    enabled: true
    max_depth: 10
  persistence:
    enabled: true
    backend: "file"
    auto_save:
      enabled: true
      interval_seconds: 60

# ── Observability (v5.1) ───────────────────────────────────────
observability:
  enabled: false
  telemetry:
    enabled: false
    service_name: "manusclaw"
  tracing:
    enabled: false
  metrics:
    enabled: false
  logging:
    structured: false
    level: "INFO"

# ── Secrets Management (v5.1) ──────────────────────────────────
secrets:
  backend: "env"
  vault:
    enabled: false
    address: "http://localhost:8200"
  encrypted_file:
    enabled: false

# ── File Store (v5.1) ──────────────────────────────────────────
file_store:
  backend: "local"
  local:
    base_path: "~/.manusclaw/files"
    max_file_size_mb: 100

# ── Git Providers (v5.1) ───────────────────────────────────────
git_providers:
  github:
    enabled: false
  gitlab:
    enabled: false
  bitbucket:
    enabled: false

# ── Integrations (v5.1) ────────────────────────────────────────
integrations:
  jira:
    enabled: false
  notion:
    enabled: false
  pagerduty:
    enabled: false
  plugins:
    enabled: true
    directory: "~/.manusclaw/plugins"

# ── Parallel Executor (v5.1) ───────────────────────────────────
parallel_executor:
  enabled: false
  mode: "threaded"
  workers:
    max_workers: 4
  retry:
    enabled: true
    max_retries: 3

# ── Migrations (v5.1) ──────────────────────────────────────────
migrations:
  auto_run: true
  backup:
    enabled: true
  rollback:
    enabled: true
```

---

## config.toml Reference

The `config.toml` file is the legacy configuration format, still fully supported in v5.1.0. New v5.1 features are configured in `config.yaml` only.

### LLM Configuration

```toml
[llm]
provider = "openai"
model = "gpt-4o"
temperature = 0.7
max_tokens = 4096
top_p = 1.0
frequency_penalty = 0.0
presence_penalty = 0.0
```

### Provider-Specific Settings

```toml
[llm.openai]
api_key = ""
organization = ""
base_url = ""

[llm.anthropic]
max_tokens = 8192

[llm.google]
max_tokens = 8192

[llm.groq]
# No special settings needed — uses GROQ_API_KEY

[llm.ollama]
base_url = "http://localhost:11434"
model = "llama3"
timeout = 300

[llm.azure]
deployment_name = "gpt-4o"
api_version = "2024-12-01-preview"

[llm.mistral]
# Uses MISTRAL_API_KEY

[llm.openrouter]
# Uses OPENROUTER_API_KEY

[llm.huggingface]
# Uses HUGGINGFACE_API_KEY

[llm.together]
# Uses TOGETHER_API_KEY

[llm.deepinfra]
# Uses DEEPINFRA_API_KEY

[llm.cohere]
# Uses COHERE_API_KEY
```

### Search Engine Configuration

```toml
[search]
engine = "duckduckgo"  # duckduckgo | google | bing | brave
max_results = 10
region = "us"
safe_search = true
```

### Token Budget Configuration

```toml
[token_budget]
max_input_tokens = 128000
max_output_tokens = 4096
max_total_tokens = 200000
auto_summarize = true
summarize_threshold = 0.8
```

### Permissions Configuration

```toml
[permissions]
mode = "PLAN"  # PLAN | AUTO | CONFIRM
auto_approve_tools = []
deny_tools = []
require_confirmation_for = ["file_write", "shell_execute", "network_access"]
```

### Workspace Configuration

```toml
[workspace]
path = "workspace"
auto_create = true
max_file_size = 10485760  # 10MB
```

### Memory Configuration

```toml
[memory]
enabled = true
max_memory_size = 10000
save_on_exit = true
file = "MEMORY.md"
user_profile = "USER.md"
```

### Logging Configuration

```toml
[logging]
level = "INFO"  # DEBUG | INFO | WARNING | ERROR | CRITICAL
file = "~/.manusclaw/logs/manusclaw.log"
max_size_mb = 50
rotation = "daily"
retention_days = 30
format = "text"  # text | json
```

### Server Configuration

```toml
[server]
host = "0.0.0.0"
port = 8765
cors_origins = ["*"]
api_key = ""  # MANUSCLAW_SERVER_API_KEY
max_request_size = 10485760
timeout = 300
```

### Sandbox Configuration

```toml
[sandbox]
enabled = false
backend = "docker"  # docker | ssh | openshell
docker_image = "python:3.12-slim"
memory_limit = "2g"
timeout = 300
network = "none"
```

---

## Environment Variables Reference

### Core Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `MANUSCLAW_CONFIG_DIR` | Custom config directory | `~/.manusclaw` |
| `MANUSCLAW_PROVIDER` | LLM provider override | From config |
| `MANUSCLAW_MODEL` | LLM model override | From config |
| `MANUSCLAW_PROFILE` | Active config profile | `default` |
| `MANUSCLAW_SERVER_API_KEY` | Server API key | None |
| `MANUSCLAW_LOG_LEVEL` | Log level override | From config |

### LLM Provider API Keys

| Variable | Provider |
|----------|----------|
| `OPENAI_API_KEY` | OpenAI |
| `OPENAI_API_KEY_1`, `_2`, `_3` | OpenAI credential pool |
| `ANTHROPIC_API_KEY` | Anthropic |
| `ANTHROPIC_API_KEY_1`, `_2` | Anthropic credential pool |
| `GOOGLE_API_KEY` | Google |
| `GROQ_API_KEY` | Groq |
| `MISTRAL_API_KEY` | Mistral |
| `OPENROUTER_API_KEY` | OpenRouter |
| `HUGGINGFACE_API_KEY` | HuggingFace |
| `TOGETHER_API_KEY` | Together AI |
| `DEEPINFRA_API_KEY` | DeepInfra |
| `COHERE_API_KEY` | Cohere |
| `AZURE_OPENAI_API_KEY` | Azure OpenAI |

### Channel API Keys

| Variable | Channel |
|----------|---------|
| `TELEGRAM_BOT_TOKEN` | Telegram |
| `DISCORD_BOT_TOKEN` | Discord |
| `SLACK_BOT_TOKEN` | Slack |
| `WHATSAPP_ACCESS_TOKEN` | WhatsApp |
| `WHATSAPP_BUSINESS_PHONE_ID` | WhatsApp |
| `SIGNAL_CLI_REST_URL` | Signal |
| `SIGNAL_CLI_NUMBER` | Signal |
| `MATRIX_HOMESERVER` | Matrix |
| `MATRIX_ACCESS_TOKEN` | Matrix |
| `MATRIX_USER_ID` | Matrix |
| `TWITCH_BOT_TOKEN` | Twitch |
| `MICROSOFT_APP_ID` | Teams |
| `MICROSOFT_APP_PASSWORD` | Teams |
| `GOOGLE_CHAT_SERVICE_ACCOUNT` | Google Chat |
| `LINE_CHANNEL_SECRET` | LINE (v5.1) |
| `LINE_CHANNEL_ACCESS_TOKEN` | LINE (v5.1) |

### Voice & SSH

| Variable | Feature |
|----------|---------|
| `PICOVOICE_API_KEY` | Voice wake word (Porcupine) |
| `ELEVENLABS_API_KEY` | Voice TTS |
| `MANUSCLAW_SSH_ENABLED` | SSH gateway |
| `MANUSCLAW_SSH_PORT` | SSH port |

### v5.1 Enterprise Variables

| Variable | Feature |
|----------|---------|
| `VAULT_ADDR` | HashiCorp Vault address |
| `VAULT_TOKEN` | Vault authentication token |
| `VAULT_ROLE_ID` | Vault AppRole role ID |
| `VAULT_SECRET_ID` | Vault AppRole secret ID |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | OpenTelemetry OTLP endpoint |
| `OTEL_SERVICE_NAME` | Telemetry service name |
| `AWS_ACCESS_KEY_ID` | AWS / S3 |
| `AWS_SECRET_ACCESS_KEY` | AWS / S3 |
| `AWS_DEFAULT_REGION` | AWS / S3 |
| `GOOGLE_APPLICATION_CREDENTIALS` | GCP / GCS / Secret Manager |
| `AZURE_STORAGE_CONNECTION_STRING` | Azure Blob Storage |
| `GITHUB_TOKEN` | GitHub integration |
| `GITLAB_TOKEN` | GitLab integration |
| `BITBUCKET_USERNAME` | Bitbucket integration |
| `BITBUCKET_APP_PASSWORD` | Bitbucket integration |
| `JIRA_API_TOKEN` | Jira integration |
| `NOTION_API_KEY` | Notion integration |
| `PAGERDUTY_API_KEY` | PagerDuty integration |

---

## Migrating from v5.0.0 to v5.1.0

### Automated Migration

```bash
# Run the migration system
manusclaw-migrate --from 5.0.0 --to 5.1.0
```

The migration system will:

1. Back up your existing configuration
2. Add new v5.1.0 sections to `config.yaml` with safe defaults
3. Create new directories (`hooks/`, `secrets/`, `migrations/`, `context/`, `conversations/`)
4. Migrate session data format if needed
5. Update `cron.yaml` format with retry policies
6. Validate the final configuration

### Manual Migration

If you prefer manual migration, add the following sections to your `config.yaml`:

```yaml
# Add these sections to your existing config.yaml:

security:
  rbac:
    enabled: false
  input_validation:
    enabled: true
  rate_limiting:
    enabled: true
  audit_logging:
    enabled: false

hooks:
  pre_execute: []
  post_execute: []
  on_error: []

context:
  compression:
    enabled: true
    strategy: "sliding"

conversation:
  threading:
    enabled: true
  persistence:
    enabled: true

observability:
  enabled: false

secrets:
  backend: "env"

file_store:
  backend: "local"

git_providers:
  github:
    enabled: false
  gitlab:
    enabled: false
  bitbucket:
    enabled: false

integrations:
  plugins:
    enabled: true

parallel_executor:
  enabled: false

migrations:
  auto_run: true
```

Then create the required directories:

```bash
mkdir -p ~/.manusclaw/{hooks,secrets,migrations,context,conversations,plugins}
```
