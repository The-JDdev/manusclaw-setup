# Configuration Guide — ManusClaw v5.0.0

ManusClaw v5.0.0 is configured through a layered system of configuration files, environment variables, and profiles. The three primary mechanisms are:

- **`config.yaml`** (v5.0.0) — Structured settings for model profiles, agent definitions and routing, voice, SSH server, and channel integrations. This is the new canonical configuration format.
- **`config.toml`** — Legacy structured settings for LLM providers, search, token budgets, permissions, workspace, memory, logging, and server. Still fully supported.
- **`.env`** — Sensitive credentials like API keys, tokens, and secrets. Never committed to version control.

Understanding how these files interact through the **7-layer config priority chain** and the **config profiles system** is essential for getting the most out of ManusClaw. This guide explains every configuration option in detail, with examples for each LLM provider and use case.

---

## Table of Contents

- [Configuration File Locations](#configuration-file-locations)
- [7-Layer Config Priority Chain](#7-layer-config-priority-chain)
- [Config Profiles](#config-profiles)
- [config.yaml Reference](#configyaml-reference)
  - [Model Profiles](#model-profiles)
  - [Agent Definitions](#agent-definitions)
  - [Agent Routes](#agent-routes)
  - [Voice Configuration](#voice-configuration)
  - [SSH Server Configuration](#ssh-server-configuration)
  - [Channels Configuration](#channels-configuration)
  - [Full Example config.yaml](#full-example-configyaml)
- [config.toml Reference](#configtoml-reference)
  - [LLM Configuration](#llm-configuration)
  - [Provider-Specific Settings](#provider-specific-settings)
  - [Search Engine Configuration](#search-engine-configuration)
  - [Token Budget Settings](#token-budget-settings)
  - [Permission Modes](#permission-modes)
  - [Workspace Configuration](#workspace-configuration)
  - [Memory Configuration](#memory-configuration)
  - [Logging Configuration](#logging-configuration)
  - [Server Configuration](#server-configuration)
  - [Cron Configuration](#cron-configuration)
  - [Full Example config.toml](#full-example-configtoml)
- [The .env File](#the-env-file)
- [LLM Provider Configuration](#llm-provider-configuration)
  - [OpenAI](#openai)
  - [Anthropic](#anthropic)
  - [Google](#google)
  - [Mistral](#mistral)
  - [Groq](#groq)
  - [AWS Bedrock](#aws-bedrock)
  - [Ollama (Local)](#ollama-local)
  - [GGUF (Local)](#gguf-local)
  - [HuggingFace](#huggingface)
  - [Universal / OpenRouter](#universal--openrouter)
- [API Key Setup](#api-key-setup)
- [Credential Pool](#credential-pool)
- [Environment Variables Reference](#environment-variables-reference)
- [Test Mode](#test-mode)

---

## Configuration File Locations

ManusClaw looks for configuration files in several locations depending on whether you are using profiles or the traditional flat layout. The locations vary slightly between `config.yaml`, `config.toml`, and `.env` files.

### Traditional (no profile) file locations

When no profile is active (the default), ManusClaw looks for configuration files in the following locations, in order of priority:

| Priority | File | Location | Description |
|----------|------|----------|-------------|
| 1 (highest) | config.toml / config.yaml | `--config` CLI flag | Explicitly specified config path |
| 2 | config.toml / config.yaml | `./config.toml` or `./config.yaml` | Current working directory |
| 3 | config.toml / config.yaml | `~/.manusclaw/config.toml` or `~/.manusclaw/config.yaml` | User's home config directory |
| 4 | config.toml / config.yaml | `/etc/manusclaw/config.toml` or `/etc/manusclaw/config.yaml` | System-wide config (Linux) |

The `.env` file follows a similar lookup pattern:

| Priority | Location |
|----------|----------|
| 1 | `--env` CLI flag |
| 2 | `./.env` |
| 3 | `~/.manusclaw/.env` |

### Profile-aware file locations

When `MANUSCLAW_PROFILE` is set, ManusClaw looks for profile-specific files first, then falls back to global files. See [Config Profiles](#config-profiles) for the full priority chain.

### First-run behavior

When ManusClaw starts for the first time, it automatically creates the `~/.manusclaw/` directory with a default `config.toml`, a starter `config.yaml`, and a `.env` file. Edit these files to match your needs.

**Why three separate files?**

- **`config.yaml`** — v5.0.0 structured settings for model profiles, agent routing, voice, SSH, and channels. Safe to commit to version control.
- **`config.toml`** — Legacy structured settings (LLM, search, workspace, memory, etc.). Safe to commit to version control.
- **`.env`** — Secrets (API keys, tokens) that should **never** be committed to a public repository. Always add `.env` to your `.gitignore`.

---

## 7-Layer Config Priority Chain

ManusClaw v5.0.0 resolves all configuration through a 7-layer priority chain. Each layer overrides the one below it. Understanding this chain is critical for predictable behavior, especially when using profiles.

| Layer | Priority | Source | Description |
|-------|----------|--------|-------------|
| 1 | **Highest** | Environment variables | Variables set in the shell or process environment. These always win. |
| 2 | | Profile `.env` | `~/.manusclaw/profiles/<name>/.env` — profile-specific secrets. |
| 3 | | Profile `config.yaml` | `~/.manusclaw/profiles/<name>/config.yaml` — profile-specific structured settings. |
| 4 | | Global `.env` | `~/.manusclaw/.env` — global secrets (fallback for missing profile secrets). |
| 5 | | Global `config.yaml` | `~/.manusclaw/config.yaml` — global structured settings (fallback). |
| 6 | | `config.toml` | `~/.manusclaw/config.toml` or CLI-specified path — legacy TOML config. |
| 7 | **Lowest** | Built-in defaults | Hardcoded defaults compiled into ManusClaw. Used when nothing else specifies a value. |

### How the priority chain works in practice

```
┌──────────────────────────────────────────────────┐
│  Layer 1: Environment variables (export FOO=bar) │  ← Always wins
├──────────────────────────────────────────────────┤
│  Layer 2: Profile .env                           │
├──────────────────────────────────────────────────┤
│  Layer 3: Profile config.yaml                    │
├──────────────────────────────────────────────────┤
│  Layer 4: Global .env                            │
├──────────────────────────────────────────────────┤
│  Layer 5: Global config.yaml                    │
├──────────────────────────────────────────────────┤
│  Layer 6: config.toml (legacy)                  │
├──────────────────────────────────────────────────┤
│  Layer 7: Built-in defaults                      │  ← Fallback
└──────────────────────────────────────────────────┘
```

**Key behaviors:**

- If you set `OPENAI_API_KEY` as an environment variable (Layer 1), it overrides the same key in any `.env` file (Layers 2, 4) or `config.toml` (Layer 6).
- If a setting exists in the profile `config.yaml` (Layer 3) but not in the global `config.yaml` (Layer 5), the profile value is used. The global value is used only when the profile file doesn't specify it.
- `config.toml` (Layer 6) is still fully supported for backward compatibility. However, for v5.0.0 features (model profiles, agents, voice, SSH, channels), you must use `config.yaml`.
- CLI flags (`--config`, `--env`) inject at the appropriate layer, effectively overriding all file-based sources at that level.

---

## Config Profiles

Config profiles allow you to maintain multiple named configuration sets and switch between them instantly. This is useful for:

- **Separating environments** — `development`, `staging`, `production`
- **Different projects** — `project-alpha`, `project-beta`
- **Different team members** — Each team member has their own profile with personal API keys
- **Different hardware** — `local-cpu`, `local-gpu`, `remote-server`

### Activating a profile

Set the `MANUSCLAW_PROFILE` environment variable:

```bash
# Activate the "production" profile
export MANUSCLAW_PROFILE=production

# Or inline
MANUSCLAW_PROFILE=staging manusclaw

# Or via .env
echo "MANUSCLAW_PROFILE=development" >> ~/.manusclaw/.env
```

### Profile directory structure

When a profile is active, ManusClaw looks for files in:

```
~/.manusclaw/
├── .env                    # Global .env (Layer 4)
├── config.yaml             # Global config.yaml (Layer 5)
├── config.toml             # Legacy config.toml (Layer 6)
└── profiles/
    ├── development/
    │   ├── .env            # Profile .env (Layer 2)
    │   └── config.yaml     # Profile config.yaml (Layer 3)
    ├── production/
    │   ├── .env
    │   └── config.yaml
    └── testing/
        ├── .env
        └── config.yaml
```

### Profile .env (Layer 2)

The profile `.env` file contains secrets specific to that profile. For example, a `production` profile might use different API keys than `development`:

```env
# ~/.manusclaw/profiles/production/.env
OPENAI_API_KEY=sk-proj-prod-key-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
ANTHROPIC_API_KEY=sk-ant-prod-key-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
MANUSCLAW_SERVER_API_KEY=prod-server-key-highly-secure
```

### Profile config.yaml (Layer 3)

The profile `config.yaml` file contains structured settings specific to that profile:

```yaml
# ~/.manusclaw/profiles/production/config.yaml
model_profiles:
  default: "production-llm"
  profiles:
    production-llm:
      provider: "openai"
      model: "gpt-4o"
      temperature: 0.2
      max_tokens: 4096

voice:
  enabled: true
  provider: "elevenlabs"
  voice_id: "professional"

server:
  host: "0.0.0.0"
  port: 8765
  cors_origins:
    - "https://your-app.example.com"
```

### Profile fallback behavior

When a profile is active but a setting is not defined in the profile's `config.yaml` or `.env`, ManusClaw falls back to the global files. This means you can:

1. Define only the settings that differ in the profile (e.g., API keys and model names)
2. Keep shared settings (e.g., workspace path, memory config) in the global files
3. Override any global setting in any profile without duplicating the entire config

### Creating a new profile

```bash
# Create the profile directory
mkdir -p ~/.manusclaw/profiles/my-profile

# Copy the global config as a starting point
cp ~/.manusclaw/config.yaml ~/.manusclaw/profiles/my-profile/config.yaml

# Create a profile-specific .env
touch ~/.manusclaw/profiles/my-profile/.env

# Edit the profile files
nano ~/.manusclaw/profiles/my-profile/config.yaml
nano ~/.manusclaw/profiles/my-profile/.env

# Activate the profile
export MANUSCLAW_PROFILE=my-profile
```

### Listing available profiles

```bash
manusclaw profile list
```

This command scans `~/.manusclaw/profiles/` and lists all profile names with their status (active/inactive).

---

## config.yaml Reference

The `config.yaml` file is the v5.0.0 canonical configuration format. It handles all features introduced in v5.0.0: model profiles, agent definitions and routing, voice, SSH server, and channel integrations. The file uses YAML format.

If a `config.yaml` file exists, ManusClaw will load it in addition to `config.toml`. Settings in `config.yaml` take precedence over `config.toml` for overlapping configuration keys (Layer 5 vs Layer 6 in the priority chain).

### Full Example config.yaml

Below is a comprehensive example showing every v5.0.0 option. You do not need to include all of these — ManusClaw uses sensible defaults for any option you omit.

```yaml
# ManusClaw Configuration File
# Version: 5.0.0

# ── Model Profiles ──────────────────────────────────────
model_profiles:
  default: "default"        # Name of the active model profile
  profiles:
    default:
      provider: "openai"
      model: "gpt-4o"
      temperature: 0.7
      max_tokens: 4096
      top_p: 1.0
      frequency_penalty: 0.0
      presence_penalty: 0.0
      timeout: 120
      retries: 3

    fast:
      provider: "openai"
      model: "gpt-4o-mini"
      temperature: 0.5
      max_tokens: 2048

    reasoning:
      provider: "anthropic"
      model: "claude-sonnet-4-20250514"
      temperature: 0.2
      max_tokens: 8192

    groq-fast:
      provider: "groq"
      model: "llama-3.1-70b-versatile"
      temperature: 0.7
      max_tokens: 4096

    local:
      provider: "ollama"
      model: "llama3"
      temperature: 0.7
      max_tokens: 4096
      timeout: 300

# ── Agent Definitions ────────────────────────────────────
agents:
  definitions:
    coder:
      name: "Coder Agent"
      description: "Specialized in writing, editing, and debugging code"
      system_prompt: |
        You are a senior software engineer. Write clean, well-documented code.
        Always explain your changes and suggest tests.
      model_profile: "default"
      temperature_override: 0.3
      max_tokens_override: 8192
      tools:
        - file_read
        - file_write
        - shell_exec
        - web_search
        - code_execute

    researcher:
      name: "Research Agent"
      description: "Specialized in research, analysis, and information gathering"
      system_prompt: |
        You are a research analyst. Thoroughly investigate topics using
        web search and provide well-structured, cited summaries.
      model_profile: "reasoning"
      tools:
        - web_search
        - web_browse
        - file_read
        - file_write
        - memory_write

    reviewer:
      name: "Code Review Agent"
      description: "Specialized in reviewing code for quality and security"
      system_prompt: |
        You are a code reviewer. Analyze code for bugs, security vulnerabilities,
        performance issues, and adherence to best practices.
      model_profile: "fast"
      tools:
        - file_read

    general:
      name: "General Assistant"
      description: "General-purpose assistant for all tasks"
      system_prompt: |
        You are ManusClaw, a helpful AI assistant. Assist the user with
        whatever they need using your available tools.
      model_profile: "default"
      tools:
        - file_read
        - file_write
        - file_delete
        - shell_exec
        - web_search
        - web_browse
        - code_execute
        - memory_write
        - skill_install

# ── Agent Routes ──────────────────────────────────────────
agents:
  routes:
    # Route by channel prefix
    - channel_prefix: "code/"
      agent: "coder"

    - channel_prefix: "research/"
      agent: "researcher"

    - channel_prefix: "review/"
      agent: "reviewer"

    # Route by channel type
    - channel_type: "irc"
      agent: "general"

    - channel_type: "twitch"
      agent: "general"

    # Route by exact channel name
    - channel: "#manusclaw-dev"
      agent: "coder"

    - channel: "#manusclaw-research"
      agent: "researcher"

    # Default fallback (must be last)
    - default: true
      agent: "general"

# ── Voice Configuration ──────────────────────────────────
voice:
  enabled: false               # Enable voice input/output
  provider: "elevenlabs"       # TTS/STT provider: elevenlabs, google, openai, azure
  voice_id: "rachel"           # Voice ID for TTS
  language: "en-US"            # Language code for STT
  speed: 1.0                   # TTS speed (0.5 - 2.0)
  pitch: 1.0                   # TTS pitch (0.5 - 2.0)
  input_device: ""             # Audio input device name (empty = system default)
  output_device: ""            # Audio output device name (empty = system default)

  # Provider-specific settings
  elevenlabs:
    api_key: ""                # Or use ELEVENLABS_API_KEY env var
    model_id: "eleven_multilingual_v2"
    stability: 0.5
    similarity_boost: 0.75

  google:
    api_key: ""                # Or use GOOGLE_TTS_API_KEY env var

  openai:
    model: "tts-1"
    voice: "alloy"

  azure:
    subscription_key: ""       # Or use AZURE_TTS_KEY env var
    region: "eastus"
    voice_name: "en-US-JennyNeural"

# ── SSH Server Configuration ─────────────────────────────
ssh_server:
  enabled: false               # Enable SSH remote access
  host: "0.0.0.0"             # Bind address
  port: 2222                  # SSH port
  host_key_path: ""           # Path to SSH host key (auto-generated if empty)
  authorized_keys_path: ""    # Path to authorized_keys file
  max_auth_tries: 3           # Maximum authentication attempts
  login_timeout: 60           # Login timeout in seconds
  allow_password_auth: false # Allow password authentication (keys recommended)
  banner: "ManusClaw v5.0.0 SSH Gateway"  # Connection banner
  username: "manusclaw"       # Required SSH username (empty = any)

# ── Channels Configuration ───────────────────────────────
channels:
  # Each channel type can be enabled/disabled independently
  # and configured with its own settings.

  # ── WhatsApp ───────────────────────────────────────
  whatsapp:
    enabled: false
    phone_number_id: ""       # From Meta Developer Dashboard
    access_token: ""          # Or use WHATSAPP_ACCESS_TOKEN env var
    verify_token: ""          # Webhook verification token
    webhook_url: ""            # Public webhook URL
    allowed_senders: []       # Phone numbers allowed to interact

  # ── Signal ──────────────────────────────────────────
  signal:
    enabled: false
    phone_number: ""           # Your Signal phone number (+1234567890)
    config_path: ""           # Signal CLI config path (auto-detected)
    allowed_senders: []       # Phone numbers allowed to interact

  # ── Matrix ─────────────────────────────────────────
  matrix:
    enabled: false
    homeserver_url: ""        # Matrix homeserver URL
    access_token: ""          # Or use MATRIX_ACCESS_TOKEN env var
    user_id: ""               # @user:homeserver.example.com
    device_id: "manusclaw"    # Device ID
    allowed_rooms: []         # Room IDs the bot will respond in
    display_name: "ManusClaw"

  # ── IRC ─────────────────────────────────────────────
  irc:
    enabled: false
    server: ""                # IRC server hostname
    port: 6697                # IRC server port (6697 = TLS)
    tls: true                 # Use TLS/SSL
    nickname: "manusclaw"     # Bot nickname
    realname: "ManusClaw Bot" # Real name
    channels:                # Channels to join
      - "#manusclaw"
      - "#general"
    nickserv_password: ""     # NickServ password for registered nick
    allowed_users: []         # IRC nicks allowed to command the bot

  # ── Twitch ──────────────────────────────────────────
  twitch:
    enabled: false
    username: ""              # Twitch bot username
    oauth_token: ""           # Or use TWITCH_OAUTH_TOKEN env var
    client_id: ""             # Or use TWITCH_CLIENT_ID env var
    client_secret: ""         # Or use TWITCH_CLIENT_SECRET env var
    channels:                # Channels to join
      - "#your_channel"
    command_prefix: "!"       # Prefix for bot commands
    rate_limit: 20            # Messages per 30 seconds (Twitch limit)

  # ── Microsoft Teams ─────────────────────────────────
  microsoft:
    enabled: false
    app_id: ""                # Or use MICROSOFT_APP_ID env var
    app_password: ""         # Or use MICROSOFT_APP_PASSWORD env var
    tenant_id: ""             # Or use MICROSOFT_TENANT_ID env var
    bot_endpoint: ""          # Bot Framework endpoint URL

  # ── Google Chat ─────────────────────────────────────
  google_chat:
    enabled: false
    service_account_json: ""  # Path to service account JSON key
    project_id: ""           # Or use GOOGLE_CHAT_PROJECT_ID env var
    subscription_name: ""     # Pub/Sub subscription name

  # ── Email (Gmail) ──────────────────────────────────
  gmail:
    enabled: false
    credentials_path: ""      # Path to OAuth credentials JSON
    token_path: ""            # Path to saved OAuth token
    watch_labels: ["INBOX"]   # Labels to watch for new emails
    allowed_senders: []       # Email addresses to respond to (empty = all)
    reply_prefix: "[ManusClaw] "  # Prefix added to replies
    max_emails_per_day: 100   # Rate limit

# ── Server (config.yaml overrides config.toml [server]) ──
server:
  host: "0.0.0.0"
  port: 8765
  workers: 1
  cors_origins:
    - "*"
  api_key: ""
```

---

### Model Profiles

Model profiles let you define multiple named LLM configurations and switch between them instantly. This is one of the most powerful v5.0.0 features — instead of editing `config.toml` every time you want to use a different model, you define named profiles and select them.

#### Why model profiles?

- **Different tasks need different models** — Use a fast model for quick tasks, a powerful model for reasoning, and a local model for privacy-sensitive work.
- **Profile-based switching** — Each config profile (see [Config Profiles](#config-profiles)) can activate a different default model profile.
- **Agent-specific models** — Each agent (see [Agent Definitions](#agent-definitions)) can use a different model profile, so your coder agent uses GPT-4o while your reviewer agent uses a fast model.

#### Defining model profiles

```yaml
model_profiles:
  default: "default"        # Which profile is active at startup
  profiles:
    default:
      provider: "openai"
      model: "gpt-4o"
      temperature: 0.7
      max_tokens: 4096

    fast:
      provider: "openai"
      model: "gpt-4o-mini"
      temperature: 0.5
      max_tokens: 2048

    reasoning:
      provider: "anthropic"
      model: "claude-sonnet-4-20250514"
      temperature: 0.2
      max_tokens: 8192

    local:
      provider: "ollama"
      model: "llama3"
      temperature: 0.7
      max_tokens: 4096
      timeout: 300
```

#### Switching model profiles at runtime

```bash
# Switch to the "fast" profile
manusclaw --model-profile fast

# Or via environment variable
export MANUSCLAW_MODEL_PROFILE=reasoning
manusclaw
```

#### Model profile fields

| Field | Type | Description | Default |
|-------|------|-------------|---------|
| `provider` | string | LLM provider name | `"openai"` |
| `model` | string | Model identifier for the provider | `"gpt-4o"` |
| `temperature` | float | Response randomness (0.0 – 2.0) | `0.7` |
| `max_tokens` | integer | Maximum tokens per response | `4096` |
| `top_p` | float | Nucleus sampling parameter | `1.0` |
| `frequency_penalty` | float | Penalize repeated tokens (0.0 – 2.0) | `0.0` |
| `presence_penalty` | float | Encourage new topics (0.0 – 2.0) | `0.0` |
| `timeout` | integer | Request timeout in seconds | `120` |
| `retries` | integer | Retry attempts on failure | `3` |

#### Integration with config.toml

If `model_profiles` is defined in `config.yaml`, it takes precedence over `[llm]` settings in `config.toml` for provider/model selection. However, provider-specific sub-sections in `config.toml` (like `[llm.openai]` for `api_key` and `base_url`) are still used for credential and endpoint configuration.

#### Using model profiles with providers that need extra config

```yaml
model_profiles:
  default: "bedrock-prod"
  profiles:
    bedrock-prod:
      provider: "bedrock"
      model: "anthropic.claude-3-5-sonnet-20241022-v2:0"
      temperature: 0.7
      max_tokens: 4096
```

The AWS credentials (access key, secret key, region) are still read from `[llm.bedrock]` in `config.toml` or from `AWS_*` environment variables.

---

### Agent Definitions

Agent definitions let you create specialized agents with custom system prompts, tool access, and model assignments. Each agent is a named entity with its own personality and capabilities.

#### Why agent definitions?

- **Specialization** — Different agents can be optimized for different tasks (coding, research, reviewing, chatting).
- **Safety** — Each agent only has access to the tools you grant it. A code reviewer can't execute shell commands.
- **Model flexibility** — Assign different model profiles to different agents based on their needs.
- **Consistent behavior** — System prompts ensure each agent behaves predictably in its domain.

#### Defining agents

```yaml
agents:
  definitions:
    coder:
      name: "Coder Agent"
      description: "Specialized in writing, editing, and debugging code"
      system_prompt: |
        You are a senior software engineer. Write clean, well-documented code.
        Always explain your changes and suggest tests.
      model_profile: "default"
      temperature_override: 0.3
      max_tokens_override: 8192
      tools:
        - file_read
        - file_write
        - shell_exec
        - web_search
        - code_execute
```

#### Agent definition fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Human-readable agent name |
| `description` | string | Yes | What the agent does |
| `system_prompt` | string | Yes | System prompt injected at conversation start |
| `model_profile` | string | No | Which model profile to use (defaults to `model_profiles.default`) |
| `temperature_override` | float | No | Override the model profile's temperature for this agent |
| `max_tokens_override` | integer | No | Override the model profile's max_tokens for this agent |
| `tools` | list | No | Tools this agent can access (subset of available tools) |

#### Available tools for agent definitions

| Tool Name | Description |
|-----------|-------------|
| `file_read` | Read file contents |
| `file_write` | Write or modify files |
| `file_delete` | Delete files |
| `shell_exec` | Execute shell commands |
| `web_search` | Search the web |
| `web_browse` | Browse web pages (Playwright) |
| `code_execute` | Execute code in a sandbox |
| `memory_write` | Write to MEMORY.md |
| `skill_install` | Install new skills |

#### Built-in agents

ManusClaw ships with a built-in `general` agent that has access to all tools. If you don't define any agents in `config.yaml`, this built-in agent is used for all requests. You can override it by defining your own agent named `general`.

---

### Agent Routes

Agent routes determine which agent handles incoming messages based on the source channel, channel prefix, or exact channel name. Routes are evaluated in order — the first matching route wins.

#### Why agent routing?

- **Channel-specific behavior** — Your IRC channel might want a general assistant, while a private Slack DM routes to a coding agent.
- **Prefix-based commands** — Users can prefix messages with `"code:"` to route to the coder agent, or `"research:"` for the researcher.
- **Room-based specialization** — Different chat rooms can be served by different agents.

#### Defining routes

```yaml
agents:
  routes:
    # Route by channel prefix (matches the beginning of the message)
    - channel_prefix: "code:"
      agent: "coder"

    - channel_prefix: "research:"
      agent: "researcher"

    # Route by channel type (matches the channel platform)
    - channel_type: "irc"
      agent: "general"

    # Route by exact channel name
    - channel: "#manusclaw-dev"
      agent: "coder"

    - channel: "#manusclaw-research"
      agent: "researcher"

    # Default fallback (must be last — matches everything unmatched)
    - default: true
      agent: "general"
```

#### Route evaluation order

Routes are evaluated top-to-bottom. The first matching route is used. The `default: true` route should always be last as a catch-all.

| Route Type | Field | Matches Against | Example |
|-----------|-------|----------------|---------|
| Prefix | `channel_prefix` | Beginning of the message text | `"code: fix the bug"` |
| Channel type | `channel_type` | Channel platform name | `"irc"`, `"whatsapp"`, `"twitch"` |
| Exact channel | `channel` | Exact channel identifier | `"#manusclaw-dev"`, `"+1234567890"` |
| Default | `default` | Always matches (catch-all) | `true` |

#### Prefix-based routing example

When a user sends a message that starts with `"code:"`, it is routed to the coder agent. The prefix is stripped before the message reaches the agent:

```
User sends: "code: fix the null pointer in auth.py"
  → Route matches: channel_prefix "code:"
  → Agent: coder
  → Agent receives: "fix the null pointer in auth.py"
```

---

### Voice Configuration

The voice system enables speech-to-text (STT) and text-to-speech (TTS) capabilities. This allows ManusClaw to process voice input and respond with synthesized speech.

#### Enabling voice

```yaml
voice:
  enabled: true
  provider: "elevenlabs"
  voice_id: "rachel"
  language: "en-US"
  speed: 1.0
```

#### Voice provider options

| Provider | STT | TTS | Notes |
|----------|-----|-----|-------|
| `elevenlabs` | No | Yes | High-quality voices, requires API key |
| `google` | Yes | Yes | Uses Google Cloud Speech API |
| `openai` | Yes | Yes | Uses OpenAI Whisper (STT) and TTS API |
| `azure` | Yes | Yes | Microsoft Azure Cognitive Services |

#### ElevenLabs configuration

```yaml
voice:
  enabled: true
  provider: "elevenlabs"
  voice_id: "rachel"
  elevenlabs:
    api_key: ""              # Or use ELEVENLABS_API_KEY env var
    model_id: "eleven_multilingual_v2"
    stability: 0.5           # 0.0 = varied, 1.0 = stable
    similarity_boost: 0.75   # Higher = more similar to original voice
```

#### Available ElevenLabs voice IDs

Popular voice IDs include: `rachel`, `drew`, `bella`, `antoni`, `elli`, `josh`, `arnold`, `sam`, `patrick`, `adam`, `callum`, `charlie`, `matilda`, `lily`, `gen`. Browse all voices at [elevenlabs.io](https://elevenlabs.io/voice-library).

---

### SSH Server Configuration

ManusClaw v5.0.0 includes an optional built-in SSH server that provides secure remote access to the ManusClaw CLI over SSH. This is useful for:

- Remote administration of your ManusClaw instance
- Team members connecting from different machines
- Integrating with terminal-based workflows
- Providing a secure, encrypted channel to ManusClaw

#### Enabling the SSH server

```yaml
ssh_server:
  enabled: true
  host: "0.0.0.0"
  port: 2222
  host_key_path: "~/.manusclaw/ssh_host_key"
  authorized_keys_path: "~/.ssh/authorized_keys"
  username: "manusclaw"
  allow_password_auth: false
  banner: "ManusClaw v5.0.0 SSH Gateway"
```

#### SSH server fields

| Field | Type | Description | Default |
|-------|------|-------------|---------|
| `enabled` | boolean | Enable/disable the SSH server | `false` |
| `host` | string | Bind address (`0.0.0.0` for all interfaces) | `"0.0.0.0"` |
| `port` | integer | SSH port number | `2222` |
| `host_key_path` | string | Path to SSH host key file (auto-generated if empty) | `""` |
| `authorized_keys_path` | string | Path to authorized_keys file | `""` |
| `max_auth_tries` | integer | Maximum authentication attempts before disconnect | `3` |
| `login_timeout` | integer | Seconds to wait for authentication | `60` |
| `allow_password_auth` | boolean | Allow password authentication (not recommended) | `false` |
| `banner` | string | Message shown when connecting | `"ManusClaw v5.0.0 SSH Gateway"` |
| `username` | string | Required SSH username (empty = any username accepted) | `""` |

#### SSH environment variables

| Variable | Description |
|----------|-------------|
| `MANUSCLAW_SSH_ENABLED` | Enable SSH server (`true`/`false`) |
| `MANUSCLAW_SSH_HOST` | Bind address |
| `MANUSCLAW_SSH_PORT` | Port number |
| `MANUSCLAW_SSH_HOST_KEY_PATH` | Path to host key |
| `MANUSCLAW_SSH_AUTHORIZED_KEYS_PATH` | Path to authorized_keys |
| `MANUSCLAW_SSH_USERNAME` | Required username |

#### Connecting to the SSH server

```bash
ssh manusclaw@your-server -p 2222

# Once connected, you get an interactive ManusClaw session:
ManusClaw v5.0.0 SSH Gateway
> Hello! How can I help you today?
```

#### Security recommendations

1. **Always use key-based authentication.** Set `allow_password_auth: false`.
2. **Restrict the bind address** to `127.0.0.1` if you don't need remote access.
3. **Use a non-standard port** to reduce automated scanning.
4. **Keep your host key secure.** Don't share `~/.manusclaw/ssh_host_key`.
5. **Use firewall rules** to limit SSH access to trusted IP addresses.

---

### Channels Configuration

Channels define how ManusClaw communicates with external messaging platforms. Each channel type is independently configurable. Enable only the channels you need.

#### Channel overview

| Channel | Direction | Use Case |
|---------|-----------|----------|
| WhatsApp | Bidirectional | Customer support, personal assistant via WhatsApp |
| Signal | Bidirectional | Privacy-focused communication via Signal |
| Matrix | Bidirectional | Self-hosted team communication |
| IRC | Bidirectional | Technical communities, open-source projects |
| Twitch | Bidirectional | Streaming, live coding, Q&A |
| Microsoft Teams | Bidirectional | Enterprise/corporate environments |
| Google Chat | Bidirectional | Google Workspace integration |
| Gmail | Receive + Reply | Email-based assistant |

#### WhatsApp channel

Requires a Meta Developer account and a registered WhatsApp Business API application.

```yaml
channels:
  whatsapp:
    enabled: true
    phone_number_id: "1234567890"
    access_token: "EAAxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    verify_token: "your-webhook-verify-token"
    webhook_url: "https://your-server.com/webhook/whatsapp"
    allowed_senders:
      - "+15551234567"
      - "+15559876543"
```

#### Signal channel

Requires Signal CLI to be installed and configured on the host machine.

```yaml
channels:
  signal:
    enabled: true
    phone_number: "+15551234567"
    config_path: "~/.config/signal"
    allowed_senders:
      - "+15559876543"
```

#### Matrix channel

```yaml
channels:
  matrix:
    enabled: true
    homeserver_url: "https://matrix.org"
    access_token: "syt_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    user_id: "@manusclaw:matrix.org"
    device_id: "manusclaw"
    allowed_rooms:
      - "!roomid:matrix.org"
    display_name: "ManusClaw"
```

#### IRC channel

```yaml
channels:
  irc:
    enabled: true
    server: "irc.libera.chat"
    port: 6697
    tls: true
    nickname: "manusclaw"
    realname: "ManusClaw Bot"
    channels:
      - "#manusclaw"
      - "#my-project"
    nickserv_password: "your-nickserv-password"
    allowed_users:
      - "my-nick"
      - "trusted-user"
```

#### Twitch channel

```yaml
channels:
  twitch:
    enabled: true
    username: "manusclaw_bot"
    oauth_token: "oauth:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    client_id: "your-client-id"
    client_secret: "your-client-secret"
    channels:
      - "#your_channel"
    command_prefix: "!"
    rate_limit: 20
```

#### Microsoft Teams channel

```yaml
channels:
  microsoft:
    enabled: true
    app_id: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    app_password: "your-app-password"
    tenant_id: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    bot_endpoint: "https://your-server.com/api/messages"
```

#### Google Chat channel

```yaml
channels:
  google_chat:
    enabled: true
    service_account_json: "/path/to/service-account-key.json"
    project_id: "your-project-id"
    subscription_name: "projects/your-project/subscriptions/manusclaw-sub"
```

#### Gmail channel

Requires OAuth 2.0 credentials for a Google Cloud project with the Gmail API enabled.

```yaml
channels:
  gmail:
    enabled: true
    credentials_path: "~/.manusclaw/gmail/credentials.json"
    token_path: "~/.manusclaw/gmail/token.json"
    watch_labels:
      - "INBOX"
      - "ManusClaw"
    allowed_senders:
      - "user@example.com"
    reply_prefix: "[ManusClaw] "
    max_emails_per_day: 100
```

---

## config.toml Reference

The `config.toml` file uses TOML format, which is a human-friendly configuration format. It handles the core LLM, search, workspace, memory, and server settings. In v5.0.0, `config.yaml` is the preferred format for new features, but `config.toml` remains fully supported for all legacy settings.

### Full Example config.toml

Below is a comprehensive example showing every option. You do not need to include all of these — ManusClaw uses sensible defaults for any option you omit.

```toml
# ManusClaw Configuration File
# Version: 5.0.0

[llm]
provider = "openai"                # Default LLM provider
model = "gpt-4o"                   # Default model name
temperature = 0.7                  # Response randomness (0.0 - 2.0)
max_tokens = 4096                  # Max tokens per response
top_p = 1.0                        # Nucleus sampling parameter
frequency_penalty = 0.0            # Penalize repeated tokens (0.0 - 2.0)
presence_penalty = 0.0             # Encourage new topics (0.0 - 2.0)
timeout = 120                      # Request timeout in seconds
retries = 3                        # Number of retry attempts on failure

[llm.openai]
api_key = ""                       # Can also use OPENAI_API_KEY env var
base_url = ""                      # Custom API endpoint (for Azure, etc.)
organization = ""                  # OpenAI organization ID

[llm.anthropic]
api_key = ""                       # Can also use ANTHROPIC_API_KEY env var
base_url = ""                      # Custom API endpoint
api_version = ""                   # API version string

[llm.google]
api_key = ""                       # Can also use GOOGLE_API_KEY env var
project_id = ""                    # Google Cloud project ID
location = "us-central1"           # Vertex AI location

[llm.mistral]
api_key = ""                       # Can also use MISTRAL_API_KEY env var
base_url = ""                      # Custom API endpoint

[llm.groq]
api_key = ""                       # Can also use GROQ_API_KEY env var
base_url = ""                      # Custom API endpoint (default: https://api.groq.com/openai/v1)

[llm.bedrock]
region = "us-east-1"               # AWS region
access_key_id = ""                 # Can also use AWS_ACCESS_KEY_ID env var
secret_access_key = ""             # Can also use AWS_SECRET_ACCESS_KEY env var
session_token = ""                 # Optional: AWS session token

[llm.ollama]
base_url = "http://localhost:11434"  # Ollama server URL
model = "llama3"                     # Default Ollama model
timeout = 300                        # Longer timeout for local inference

[llm.gguf]
model_path = ""                    # Path to .gguf model file
n_ctx = 4096                       # Context window size
n_gpu_layers = 0                   # GPU layers (0 = CPU only, -1 = all)
n_threads = 4                      # CPU threads for inference

[llm.huggingface]
api_key = ""                       # Can also use HUGGINGFACE_API_KEY env var
model = ""                         # HuggingFace model ID
base_url = "https://api-inference.huggingface.co"

[llm.universal]
api_key = ""                       # OpenRouter or compatible API key
base_url = "https://openrouter.ai/api/v1"
model = "openai/gpt-4o"           # Model in provider/model format

[search]
engine = "duckduckgo"              # Search engine: duckduckgo, google, bing
max_results = 10                   # Maximum search results per query
region = "wt-wt"                   # Search region (wt-wt = worldwide)
safe_search = "moderate"           # Safe search: off, moderate, strict

[search.google]
api_key = ""                       # Google Custom Search API key
cx = ""                            # Custom Search Engine ID

[search.bing]
api_key = ""                       # Bing Search API key

[token_budget]
max_input_tokens = 128000          # Maximum input token budget per request
max_output_tokens = 4096           # Maximum output tokens per response
max_total_tokens = 200000          # Total token budget per conversation
warning_threshold = 0.8            # Warn when this fraction of budget is used
auto_summarize = true              # Auto-summarize conversation when budget is low

[permissions]
mode = "PLAN"                      # PLAN or BUILD mode (see below)
auto_approve = []                  # Tools that don't require confirmation
deny = []                          # Tools that are always blocked
require_confirmation = ["file_write", "shell_exec"]  # Tools requiring user OK

[workspace]
path = "workspace"                 # Default workspace path (relative or absolute)
auto_create = true                 # Auto-create workspace if it doesn't exist
git_init = true                    # Initialize git repo in workspace

[memory]
enabled = true                     # Enable/disable the memory system
memory_file = "MEMORY.md"         # Path to the persistent memory file
user_file = "USER.md"             # Path to the user profile file
max_memory_size = 10000           # Maximum characters in memory file
auto_save = true                  # Automatically save memory after conversations
save_interval = 300               # Auto-save interval in seconds

[logging]
level = "INFO"                     # Log level: DEBUG, INFO, WARNING, ERROR
file = ""                          # Log file path (empty = no file logging)
rotation = "10 MB"                 # Log file rotation size
retention = "7 days"              # How long to keep log files
format = "{time:YYYY-MM-DD HH:mm:ss} | {level} | {message}"  # Log format

[server]
host = "0.0.0.0"                  # Server bind address
port = 8765                        # Server port (changed from 8000 in v5.0.0)
workers = 1                        # Number of uvicorn workers
cors_origins = ["*"]              # CORS allowed origins
api_key = ""                       # API key for server authentication

[cron]
enabled = false                    # Enable the cron scheduler
timezone = "UTC"                   # Timezone for cron schedules
```

---

### LLM Configuration

The `[llm]` section controls which LLM provider and model ManusClaw uses by default. When `model_profiles` is defined in `config.yaml`, the active model profile takes precedence. Otherwise, the `[llm]` section is used directly.

```toml
[llm]
provider = "openai"      # Which provider to use
model = "gpt-4o"         # Which model to request
temperature = 0.7        # How creative/random the responses are
max_tokens = 4096        # Maximum tokens in the model's response
```

**`provider`** — Determines which LLM service ManusClaw communicates with. Valid values are:

| Value | Service | Requires API Key |
|-------|---------|-----------------|
| `"openai"` | OpenAI API | Yes |
| `"anthropic"` | Anthropic API (Claude) | Yes |
| `"google"` | Google AI / Vertex AI | Yes |
| `"mistral"` | Mistral AI API | Yes |
| `"groq"` | Groq (low-latency inference) | Yes |
| `"bedrock"` | AWS Bedrock | Yes (AWS credentials) |
| `"ollama"` | Ollama (local) | No |
| `"gguf"` | Local GGUF file | No |
| `"huggingface"` | HuggingFace Inference API | Yes |
| `"universal"` | OpenRouter / any OpenAI-compatible API | Yes |

**`model`** — The model identifier to use. The exact string depends on the provider. Examples:

- OpenAI: `"gpt-4o"`, `"gpt-4o-mini"`, `"gpt-4-turbo"`, `"o1-preview"`
- Anthropic: `"claude-sonnet-4-20250514"`, `"claude-3-5-haiku-20241022"`, `"claude-3-opus-20240229"`
- Google: `"gemini-2.0-flash"`, `"gemini-1.5-pro"`
- Mistral: `"mistral-large-latest"`, `"mistral-medium-latest"`, `"codestral-latest"`
- Groq: `"llama-3.1-70b-versatile"`, `"llama-3.1-8b-instant"`, `"mixtral-8x7b-32768"`
- Ollama: `"llama3"`, `"mistral"`, `"codellama"`, `"phi3"`
- OpenRouter: `"openai/gpt-4o"`, `"anthropic/claude-3.5-sonnet"`

**`temperature`** — Controls response randomness. A value of `0.0` produces deterministic, focused responses (best for coding tasks), while `2.0` produces highly creative, varied responses (best for brainstorming). The default of `0.7` is a good balance for most use cases. When using ManusClaw for code generation or precise task execution, a lower temperature (0.0–0.3) is recommended.

**`max_tokens`** — The maximum number of tokens the model will generate in a single response. Higher values allow longer responses but cost more and take longer. For most tasks, 4096 is sufficient. For long-form content generation, you may want to increase this to 8192 or higher (if the model supports it).

---

### Provider-Specific Settings

Each provider has its own subsection under `[llm]` where you can set provider-specific configuration. These settings override the general `[llm]` settings when that provider is active.

See the [LLM Provider Configuration](#llm-provider-configuration) section below for detailed setup instructions for each provider.

---

### Search Engine Configuration

ManusClaw includes built-in web search capabilities, which are essential for tasks that require up-to-date information. The search system is configured in the `[search]` section.

```toml
[search]
engine = "duckduckgo"     # Primary search engine
max_results = 10          # Max results per query
region = "wt-wt"          # Geographic region for results
safe_search = "moderate"  # Content filtering level
```

**`engine`** — The search engine to use. Options:

- `"duckduckgo"` (default) — No API key required. Uses DuckDuckGo's HTML search. This is the easiest option and works out of the box.
- `"google"` — Requires a Google Custom Search API key and a Custom Search Engine ID. Provides higher-quality results and more reliable access.
- `"bing"` — Requires a Bing Search API key from Azure.

**Why DuckDuckGo?** DuckDuckGo search works without any API key or configuration, making it the best default choice. However, it can be rate-limited or blocked if you make too many requests in a short period. For production or heavy use, Google or Bing search APIs are more reliable.

**Google Custom Search setup:**

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a project and enable the "Custom Search API"
3. Create credentials (API key)
4. Go to [programmablesearchengine.google.com](https://programmablesearchengine.google.com/) and create a search engine
5. Note the Search Engine ID (cx)

```toml
[search.google]
api_key = "AIzaSy..."     # Your Google API key
cx = "a1b2c3..."          # Your Custom Search Engine ID
```

---

### Token Budget Settings

Token budgets prevent conversations from growing too large for the model's context window, which would cause errors or truncated responses. The budget system monitors token usage and can automatically summarize older conversation history when the budget is running low.

```toml
[token_budget]
max_input_tokens = 128000     # Maximum tokens in the input/context
max_output_tokens = 4096      # Maximum tokens in the model's response
max_total_tokens = 200000     # Total budget per conversation
warning_threshold = 0.8       # Warn at 80% usage
auto_summarize = true         # Automatically summarize when budget is tight
```

**`max_input_tokens`** — This should generally match the context window of your chosen model. Common context windows:

| Model | Context Window | Recommended Setting |
|-------|---------------|-------------------|
| GPT-4o | 128K | `128000` |
| GPT-4o-mini | 128K | `128000` |
| Claude 3.5 Sonnet | 200K | `200000` |
| Claude 3 Opus | 200K | `200000` |
| Gemini 1.5 Pro | 1M | `1000000` |
| Llama 3 (8B) | 8K | `8000` |
| Llama 3.1 (70B) | 128K | `128000` |

**`warning_threshold`** — When the conversation's token usage reaches this fraction of the total budget, ManusClaw will warn you and (if `auto_summarize` is enabled) begin summarizing older messages to free up space.

**`auto_summarize`** — When enabled, ManusClaw automatically summarizes older conversation turns when the token budget is running low, preserving the most recent and relevant context while reducing total token count. This is highly recommended for long work sessions.

---

### Permission Modes

ManusClaw has two permission modes that control how aggressively the agent can execute actions. This is a critical safety feature that prevents unintended consequences.

```toml
[permissions]
mode = "PLAN"                             # PLAN or BUILD
auto_approve = []                         # Always-allowed tools
deny = []                                 # Always-blocked tools
require_confirmation = ["file_write", "shell_exec"]  # Need user OK
```

**`mode = "PLAN"`** (default, safer):
- The agent proposes a plan of action but **does not execute** it
- Every action that modifies files, runs commands, or accesses external services requires your explicit approval
- Best for: exploring, learning, reviewing code, or any task where you want full oversight
- You will see: "I plan to: [action]. Proceed? (y/n)"

**`mode = "BUILD"`** (more autonomous):
- The agent executes actions directly without asking for confirmation
- Only tools in the `require_confirmation` list will prompt for approval
- Best for: well-defined tasks, CI/CD pipelines, server automation, or when you trust the agent's judgment
- **Warning:** In BUILD mode, the agent can delete files, run arbitrary commands, and make changes without asking. Always review your `deny` list.

**Tool permissions explained:**

The `auto_approve`, `deny`, and `require_confirmation` lists contain tool names that override the mode-based defaults:

- **`auto_approve`** — These tools are always allowed, regardless of mode. Example: adding `"file_read"` means the agent can always read files without asking.
- **`deny`** — These tools are always blocked. Example: adding `"shell_exec"` prevents the agent from ever running shell commands.
- **`require_confirmation`** — These tools always require your approval, even in BUILD mode.

Common tool names for permission lists:

| Tool Name | Description |
|-----------|-------------|
| `file_read` | Read file contents |
| `file_write` | Write or modify files |
| `file_delete` | Delete files |
| `shell_exec` | Execute shell commands |
| `web_search` | Search the web |
| `web_browse` | Browse web pages (Playwright) |
| `code_execute` | Execute code in a sandbox |
| `memory_write` | Write to MEMORY.md |
| `skill_install` | Install new skills |

---

### Workspace Configuration

The workspace is the directory where ManusClaw operates — where it reads, writes, and executes tasks. Think of it as the agent's working directory.

```toml
[workspace]
path = "workspace"        # Relative to where manusclaw is launched, or absolute
auto_create = true        # Create workspace if missing
git_init = true           # Initialize git repo for version tracking
```

**`path`** — Can be a relative path (resolved from where you launch `manusclaw`) or an absolute path. Can also be set via `MANUSCLAW_WORKSPACE` environment variable. Examples:

```toml
# Relative path (created in current directory)
path = "workspace"

# Absolute path
path = "/home/user/projects/my-app"

# Windows path (use forward slashes even on Windows)
path = "C:/Users/user/projects/my-app"
```

**`git_init`** — When true, ManusClaw initializes a git repository in the workspace. This provides built-in version control so you can review and revert any changes the agent makes. This is highly recommended — it gives you a safety net for any modifications the agent performs.

---

### Memory Configuration

The memory system allows ManusClaw to persist context across sessions. This includes facts about your project, your preferences, and ongoing task progress.

```toml
[memory]
enabled = true                # Enable persistent memory
memory_file = "MEMORY.md"    # Agent's memory file
user_file = "USER.md"        # User profile/preferences
max_memory_size = 10000      # Max characters in MEMORY.md
auto_save = true             # Auto-save after conversations
save_interval = 300          # Auto-save every 5 minutes
```

The memory system uses two Markdown files:

- **MEMORY.md** — The agent's memory. It stores facts about your project, codebase structure, decisions made, and any information the agent needs to remember across conversations. The agent reads this file at the start of each session and updates it as needed.
- **USER.md** — Your profile and preferences. This includes your coding style, preferred frameworks, communication preferences, and any personal context you want the agent to know.

---

### Logging Configuration

```toml
[logging]
level = "INFO"               # DEBUG, INFO, WARNING, ERROR
file = ""                    # Empty = console only; set path for file logging
rotation = "10 MB"           # Rotate log file when it reaches this size
retention = "7 days"         # Delete logs older than this
format = "{time:YYYY-MM-DD HH:mm:ss} | {level} | {message}"
```

**`level`** — Controls verbosity:

- `DEBUG` — Extremely verbose; shows every internal operation. Useful for troubleshooting.
- `INFO` — Normal operation; shows startup messages, tool calls, and important events.
- `WARNING` — Only shows potential problems and errors.
- `ERROR` — Only shows errors that prevent normal operation.

**`file`** — When set to a file path, ManusClaw writes logs to that file in addition to the console. This is essential for server deployments where you need to review logs after the fact.

```toml
# Example: Log to file
file = "/var/log/manusclaw/manusclaw.log"

# Example: Log to user directory
file = "~/.manusclaw/manusclaw.log"
```

---

### Server Configuration

When running ManusClaw in server mode (`manusclaw-server`), these settings control the HTTP server behavior.

> **⚠️ Port change in v5.0.0:** The default server port changed from `8000` to `8765`. Update any firewall rules, reverse proxy configs, or client connections accordingly.

```toml
[server]
host = "0.0.0.0"           # Bind address (0.0.0.0 = all interfaces)
port = 8765                  # Server port (changed from 8000 in v5.0.0)
workers = 1                 # Uvicorn worker processes
cors_origins = ["*"]       # Allowed CORS origins
api_key = ""                # API key for server authentication
```

**`host`** — The network interface to bind to. Use `"0.0.0.0"` to accept connections from any IP (required for remote access), or `"127.0.0.1"` to only accept local connections (more secure).

**`api_key`** — When set, all API requests to the server must include this key in the `Authorization` header. This is critical for security when exposing the server to the internet:

```bash
# With API key (note the new port 8765)
curl -H "Authorization: Bearer your-api-key" http://localhost:8765/api/chat
```

**`cors_origins`** — Controls which domains can make browser-based requests to the server. Use `["*"]` for development, but specify exact domains in production:

```toml
cors_origins = ["https://your-app.example.com", "https://admin.example.com"]
```

**`MANUSCLAW_ALLOWED_ORIGINS` environment variable:**

You can also set CORS origins via the `MANUSCLAW_ALLOWED_ORIGINS` environment variable. This takes precedence over the `cors_origins` field in `config.toml`:

```bash
export MANUSCLAW_ALLOWED_ORIGINS="https://app.example.com,https://admin.example.com"
```

---

### Cron Configuration

ManusClaw includes a built-in cron scheduler for recurring tasks:

```toml
[cron]
enabled = false             # Enable/disable cron scheduling
timezone = "UTC"            # Timezone for schedule interpretation
```

In v5.0.0, the cron file path can be customized via the `MANUSCLAW_CRON_FILE` environment variable:

```bash
# Default: ~/.manusclaw/cron.yaml
export MANUSCLAW_CRON_FILE="/etc/manusclaw/cron.yaml"
```

Cron tasks are configured separately using the `manusclaw-cron` command. See the [Usage Guide](usage.md) for details.

---

## The .env File

The `.env` file stores environment variables, primarily API keys and other sensitive credentials. It is loaded automatically when ManusClaw starts.

### Full .env file example (v5.0.0)

```env
# ── LLM Provider Keys ──────────────────────────────────
# OpenAI
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Anthropic
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Google
GOOGLE_API_KEY=AIzaSyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Mistral
MISTRAL_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Groq
GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# AWS Bedrock
AWS_ACCESS_KEY_ID=AKIAxxxxxxxxxxxxxxxx
AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AWS_SESSION_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AWS_REGION=us-east-1

# HuggingFace
HUGGINGFACE_API_KEY=hf_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# OpenRouter / Universal
OPENROUTER_API_KEY=sk-or-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ── Search Provider Keys ──────────────────────────────
GOOGLE_SEARCH_API_KEY=AIzaSyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
GOOGLE_SEARCH_CX=xxxxxxxxxxxxxxxxxxxxx
BING_SEARCH_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ── Groq ────────────────────────────────────────────────
GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ── Server ──────────────────────────────────────────────
MANUSCLAW_SERVER_API_KEY=your-secure-api-key-here
MANUSCLAW_ALLOWED_ORIGINS=https://your-app.example.com

# ── Voice ───────────────────────────────────────────────
ELEVENLABS_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ── WhatsApp Channel ───────────────────────────────────
WHATSAPP_ACCESS_TOKEN=EAAxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
WHATSAPP_PHONE_NUMBER_ID=1234567890
WHATSAPP_VERIFY_TOKEN=your-webhook-verify-token

# ── Signal Channel ─────────────────────────────────────
SIGNAL_PHONE_NUMBER=+15551234567

# ── Matrix Channel ─────────────────────────────────────
MATRIX_ACCESS_TOKEN=syt_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ── IRC Channel ─────────────────────────────────────────
IRC_NICKSERV_PASSWORD=your-nickserv-password

# ── Twitch Channel ─────────────────────────────────────
TWITCH_OAUTH_TOKEN=oauth:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWITCH_CLIENT_ID=your-client-id
TWITCH_CLIENT_SECRET=your-client-secret

# ── Microsoft Teams Channel ────────────────────────────
MICROSOFT_APP_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
MICROSOFT_APP_PASSWORD=your-app-password
MICROSOFT_TENANT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

# ── Google Chat Channel ────────────────────────────────
GOOGLE_CHAT_PROJECT_ID=your-project-id

# ── Gmail Channel ─────────────────────────────────────
GMAIL_CREDENTIALS_PATH=~/.manusclaw/gmail/credentials.json
GMAIL_TOKEN_PATH=~/.manusclaw/gmail/token.json

# ── SSH Server ─────────────────────────────────────────
MANUSCLAW_SSH_ENABLED=false
MANUSCLAW_SSH_HOST=0.0.0.0
MANUSCLAW_SSH_PORT=2222
MANUSCLAW_SSH_HOST_KEY_PATH=~/.manusclaw/ssh_host_key
MANUSCLAW_SSH_AUTHORIZED_KEYS_PATH=~/.ssh/authorized_keys
MANUSCLAW_SSH_USERNAME=manusclaw

# ── Sandbox ────────────────────────────────────────────
SANDBOX_BACKEND=docker                    # docker, podman, firejail, none
SSH_SANDBOX_HOST=localhost
SSH_SANDBOX_PORT=2223
SSH_SANDBOX_IMAGE=manusclaw/sandbox:latest

# ── Image Generation ───────────────────────────────────
FAL_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ── Cron ───────────────────────────────────────────────
MANUSCLAW_CRON_FILE=~/.manusclaw/cron.yaml

# ── Profiles ───────────────────────────────────────────
MANUSCLAW_PROFILE=                          # Active profile name (empty = no profile)

# ── Redaction ──────────────────────────────────────────
MANUSCLAW_REDACT=false                      # Redact sensitive data in logs

# ── Environment ────────────────────────────────────────
APP_ENV=production                          # production, development, test
MANUSCLAW_HOME=~/.manusclaw                  # ManusClaw home directory
MANUSCLAW_WORKSPACE=workspace               # Default workspace path
MANUSCLAW_SKILLS_DIR=~/.manusclaw/skills    # Skills directory
```

### How .env variables interact with config files

Environment variables take precedence over values in `config.toml` and `config.yaml`. This means:

1. If you set `OPENAI_API_KEY` in `.env`, it overrides `api_key` in `[llm.openai]` in `config.toml`
2. If you set both an env var and a config file value, the environment variable wins
3. This allows you to have a shared `config.toml`/`config.yaml` while keeping secrets in `.env`

### Security best practices for .env

1. **Never commit `.env` to version control.** Add it to `.gitignore`:
   ```bash
   echo ".env" >> ~/.manusclaw/.gitignore
   ```

2. **Set restrictive file permissions:**
   ```bash
   chmod 600 ~/.manusclaw/.env
   ```

3. **Use the minimum necessary permissions** when creating API keys. For example, if an API key only needs read access, don't give it write access.

4. **Rotate keys regularly.** If a key is compromised, revoke it immediately and generate a new one.

5. **Don't share keys between environments.** Use different API keys for development, staging, and production. Config profiles make this easy — each profile can have its own `.env`.

6. **Use `MANUSCLAW_REDACT=true`** in production to automatically redact API keys and sensitive data from log output.

---

## LLM Provider Configuration

Each LLM provider requires slightly different configuration. This section covers every supported provider with complete setup instructions.

### OpenAI

OpenAI provides some of the most capable models available, including GPT-4o and the o1 reasoning series. This is the most commonly used provider with ManusClaw.

**Getting an API key:**
1. Go to [platform.openai.com](https://platform.openai.com/)
2. Sign up or log in
3. Navigate to API Keys → "Create new secret key"
4. Copy the key immediately (you won't be able to see it again)

**Configuration:**

```toml
[llm]
provider = "openai"
model = "gpt-4o"
temperature = 0.7
max_tokens = 4096

[llm.openai]
api_key = ""                       # Or use OPENAI_API_KEY env var
base_url = ""                      # Leave empty for standard OpenAI API
organization = ""                  # Optional: for organization accounts
```

**.env file:**
```env
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**Using a custom base URL (Azure OpenAI, etc.):**

If you're using Azure OpenAI Service or another OpenAI-compatible API, set the `base_url`:

```toml
[llm.openai]
base_url = "https://your-resource.openai.azure.com/openai/deployments/your-deployment"
api_key = "your-azure-api-key"
```

For Azure, you also need to set the API version:

```env
OPENAI_API_VERSION=2024-02-15-preview
```

**Recommended models:**

| Model | Context | Best For | Relative Cost |
|-------|---------|----------|--------------|
| gpt-4o | 128K | General-purpose, coding, analysis | $$$ |
| gpt-4o-mini | 128K | Quick tasks, high-volume usage | $ |
| o1-preview | 128K | Complex reasoning, math, science | $$$$ |
| o1-mini | 128K | Reasoning tasks (smaller model) | $$ |
| gpt-4-turbo | 128K | Legacy (use gpt-4o instead) | $$$ |

---

### Anthropic

Anthropic's Claude models are known for their strong reasoning capabilities, helpfulness, and ability to follow complex instructions. Claude 3.5 Sonnet is particularly well-suited for coding tasks.

**Getting an API key:**
1. Go to [console.anthropic.com](https://console.anthropic.com/)
2. Sign up or log in
3. Navigate to API Keys → "Create Key"
4. Copy the key

**Configuration:**

```toml
[llm]
provider = "anthropic"
model = "claude-sonnet-4-20250514"
temperature = 0.7
max_tokens = 4096

[llm.anthropic]
api_key = ""                       # Or use ANTHROPIC_API_KEY env var
base_url = ""                      # Optional custom endpoint
```

**.env file:**
```env
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**Recommended models:**

| Model | Context | Best For | Relative Cost |
|-------|---------|----------|--------------|
| claude-sonnet-4-20250514 | 200K | Best balance of speed, intelligence, and cost | $$ |
| claude-3-5-haiku-20241022 | 200K | Fast, affordable responses | $ |
| claude-3-opus-20240229 | 200K | Most capable, complex reasoning | $$$$ |

---

### Google

Google provides the Gemini family of models, including Gemini 2.0 Flash (fast) and Gemini 1.5 Pro (with a massive 1M token context window). You can use either the Google AI API (simpler) or Vertex AI (enterprise).

**Getting an API key (Google AI Studio):**
1. Go to [aistudio.google.com](https://aistudio.google.com/)
2. Click "Get API Key"
3. Create or select a Google Cloud project
4. Copy the API key

**Configuration:**

```toml
[llm]
provider = "google"
model = "gemini-2.0-flash"
temperature = 0.7
max_tokens = 4096

[llm.google]
api_key = ""                       # Or use GOOGLE_API_KEY env var
project_id = ""                    # Required for Vertex AI
location = "us-central1"           # Required for Vertex AI
```

**.env file:**
```env
GOOGLE_API_KEY=AIzaSyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**Vertex AI (enterprise):**

If you're using Vertex AI instead of the Google AI API, you need:

```env
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-central1
```

And authenticate using the gcloud CLI:

```bash
gcloud auth application-default login
```

**Recommended models:**

| Model | Context | Best For | Relative Cost |
|-------|---------|----------|--------------|
| gemini-2.0-flash | 1M | Fast, versatile, great value | $ |
| gemini-1.5-pro | 1M | Complex reasoning, long documents | $$ |

---

### Mistral

Mistral AI offers competitive open and commercial models. Their models are particularly good at coding tasks and multilingual applications.

**Getting an API key:**
1. Go to [console.mistral.ai](https://console.mistral.ai/)
2. Sign up or log in
3. Navigate to API Keys → "Create new key"
4. Copy the key

**Configuration:**

```toml
[llm]
provider = "mistral"
model = "mistral-large-latest"
temperature = 0.7
max_tokens = 4096

[llm.mistral]
api_key = ""                       # Or use MISTRAL_API_KEY env var
base_url = ""                      # Optional custom endpoint
```

**.env file:**
```env
MISTRAL_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**Recommended models:**

| Model | Context | Best For | Relative Cost |
|-------|---------|----------|--------------|
| mistral-large-latest | 128K | Most capable, complex tasks | $$$ |
| mistral-medium-latest | 32K | Balanced performance and cost | $$ |
| mistral-small-latest | 32K | Fast, affordable | $ |
| codestral-latest | 32K | Code generation and review | $$ |
| open-mistral-nemo | 128K | Open-weight, cost-effective | $ |

---

### Groq

Groq provides extremely fast LLM inference using their LPU (Language Processing Unit) hardware. This makes Groq ideal for use cases where low latency is critical, such as real-time chat, voice assistants, and interactive coding help.

**Getting an API key:**
1. Go to [console.groq.com](https://console.groq.com/)
2. Sign up or log in
3. Navigate to API Keys → "Create Key"
4. Copy the key

**Configuration:**

```toml
[llm]
provider = "groq"
model = "llama-3.1-70b-versatile"
temperature = 0.7
max_tokens = 4096

[llm.groq]
api_key = ""                       # Or use GROQ_API_KEY env var
base_url = ""                      # Optional (default: https://api.groq.com/openai/v1)
```

**.env file:**
```env
GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**Using Groq with model profiles (recommended):**

```yaml
model_profiles:
  default: "groq-fast"
  profiles:
    groq-fast:
      provider: "groq"
      model: "llama-3.1-70b-versatile"
      temperature: 0.7
      max_tokens: 4096
```

**Recommended models:**

| Model | Context | Best For | Speed |
|-------|---------|----------|-------|
| llama-3.1-70b-versatile | 128K | General-purpose, coding, analysis | ~300 tok/s |
| llama-3.1-8b-instant | 128K | Quick tasks, high-volume | ~500+ tok/s |
| mixtral-8x7b-32768 | 32K | Balanced performance | ~200 tok/s |
| gemma2-9b-it | 8K | Lightweight tasks | ~400+ tok/s |

---

### AWS Bedrock

AWS Bedrock provides access to multiple foundation models through a single API, including Claude, Llama, Mistral, and Titan models. It's ideal for organizations already using AWS.

**Getting credentials:**
1. Go to the [AWS IAM Console](https://console.aws.amazon.com/iam/)
2. Create a new IAM user or use an existing one
3. Attach the `AmazonBedrockFullAccess` policy (or a more restrictive custom policy)
4. Create access keys for the user

**Configuration:**

```toml
[llm]
provider = "bedrock"
model = "anthropic.claude-3-5-sonnet-20241022-v2:0"
temperature = 0.7
max_tokens = 4096

[llm.bedrock]
region = "us-east-1"               # AWS region where Bedrock is available
access_key_id = ""                  # Or use AWS_ACCESS_KEY_ID env var
secret_access_key = ""              # Or use AWS_SECRET_ACCESS_KEY env var
session_token = ""                  # Optional: for temporary credentials
```

**.env file:**
```env
AWS_ACCESS_KEY_ID=AKIAxxxxxxxxxxxxxxxx
AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AWS_SESSION_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AWS_REGION=us-east-1
```

**Important:** You must enable the specific models you want to use in the AWS Bedrock console before you can access them. Go to "Model Access" in the Bedrock console and request access to each model.

**Recommended models:**

| Model ID | Best For |
|----------|----------|
| anthropic.claude-3-5-sonnet-20241022-v2:0 | General-purpose, coding |
| anthropic.claude-3-haiku-20240307-v1:0 | Fast, affordable |
| meta.llama3-1-70b-instruct-v1:0 | Open-weight, cost-effective |
| mistral.mistral-large-2407-v1:0 | Complex reasoning |

---

### Ollama (Local)

Ollama allows you to run LLMs locally on your own hardware. This is completely free, private, and works without an internet connection (after downloading models). It's the best option for privacy-conscious users or those who want to avoid API costs.

**Installing Ollama:**

```bash
# Linux
curl -fsSL https://ollama.com/install.sh | sh

# macOS
brew install ollama

# Windows
# Download from https://ollama.com/download/windows
```

**Starting the Ollama server:**

```bash
# Start the server (it runs in the background)
ollama serve

# In another terminal, pull a model
ollama pull llama3
ollama pull codellama
ollama pull mistral
```

**Configuration:**

```toml
[llm]
provider = "ollama"
model = "llama3"
temperature = 0.7
max_tokens = 4096

[llm.ollama]
base_url = "http://localhost:11434"   # Ollama API URL
model = "llama3"                       # Model name (must be pulled first)
timeout = 300                          # Local inference can be slow
```

**.env file:**
```env
# No API key needed for Ollama!
# But if Ollama is on a remote server:
OLLAMA_BASE_URL=http://192.168.1.100:11434
```

**Remote Ollama server:**

If Ollama is running on a different machine (e.g., a GPU server), point ManusClaw to it:

```toml
[llm.ollama]
base_url = "http://192.168.1.100:11434"
```

Make sure the Ollama server is configured to accept connections from external IPs:

```bash
# On the Ollama server, set the host environment variable
OLLAMA_HOST=0.0.0.0 ollama serve
```

**Hardware requirements for local models:**

| Model | Minimum RAM | Recommended RAM | GPU |
|-------|------------|----------------|-----|
| llama3 (8B) | 8 GB | 16 GB | Optional |
| mistral (7B) | 8 GB | 16 GB | Optional |
| codellama (13B) | 16 GB | 32 GB | Recommended |
| llama3 (70B) | 40 GB | 64 GB | Required |
| mixtral (8x7B) | 32 GB | 64 GB | Required |

---

### GGUF (Local)

GGUF is a file format for quantized models that can run locally using llama.cpp, which is bundled with ManusClaw. This gives you the most control over local model inference, including GPU offloading and fine-grained parameter tuning.

**Getting GGUF models:**

1. Visit [huggingface.co/models?search=gguf](https://huggingface.co/models?search=gguf)
2. Find a model you want (e.g., "Meta-Llama-3-8B-Instruct-GGUF")
3. Download a quantized version (Q4_K_M is a good balance of quality and size)
4. Save it to a known location

**Configuration:**

```toml
[llm]
provider = "gguf"
model = "llama-3-8b-instruct.Q4_K_M.gguf"

[llm.gguf]
model_path = "/home/user/models/llama-3-8b-instruct.Q4_K_M.gguf"
n_ctx = 4096                        # Context window size
n_gpu_layers = 0                    # 0 = CPU only, -1 = all layers on GPU
n_threads = 4                       # CPU threads
```

**GPU acceleration:**

To use GPU acceleration, set `n_gpu_layers` to a positive number or `-1` (all layers):

```toml
[llm.gguf]
n_gpu_layers = -1                   # Use GPU for all layers
```

This requires a compatible GPU (NVIDIA with CUDA, Apple Metal, or AMD with ROCm) and the appropriate build of llama.cpp.

---

### HuggingFace

HuggingFace provides inference APIs for thousands of models. This is useful for accessing models that aren't available through other providers.

**Getting an API key:**
1. Go to [huggingface.co](https://huggingface.co/)
2. Sign up or log in
3. Go to Settings → Access Tokens → "New token"
4. Copy the token

**Configuration:**

```toml
[llm]
provider = "huggingface"
model = "meta-llama/Meta-Llama-3-8B-Instruct"

[llm.huggingface]
api_key = ""                       # Or use HUGGINGFACE_API_KEY env var
base_url = "https://api-inference.huggingface.co"
```

**.env file:**
```env
HUGGINGFACE_API_KEY=hf_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

### Universal / OpenRouter

The "Universal" provider connects to any OpenAI-compatible API, with OpenRouter being the most popular option. OpenRouter provides a unified API for hundreds of models from different providers, often at competitive prices.

**Getting an OpenRouter API key:**
1. Go to [openrouter.ai](https://openrouter.ai/)
2. Sign up or log in
3. Navigate to Keys → "Create Key"
4. Copy the key

**Configuration:**

```toml
[llm]
provider = "universal"
model = "openai/gpt-4o"

[llm.universal]
api_key = ""                       # Or use OPENROUTER_API_KEY env var
base_url = "https://openrouter.ai/api/v1"
model = "openai/gpt-4o"           # Format: provider/model
```

**.env file:**
```env
OPENROUTER_API_KEY=sk-or-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**Using other OpenAI-compatible APIs:**

Any service that provides an OpenAI-compatible API endpoint can be used with the Universal provider:

```toml
[llm.universal]
# Together AI
base_url = "https://api.together.xyz/v1"
api_key = "your-together-api-key"
model = "meta-llama/Llama-3-70b-chat-hf"

# Groq (also available as native provider in v5.0.0)
base_url = "https://api.groq.com/openai/v1"
api_key = "your-groq-api-key"
model = "llama-3.1-70b-versatile"

# Fireworks AI
base_url = "https://api.fireworks.ai/inference/v1"
api_key = "your-fireworks-api-key"
model = "accounts/fireworks/models/llama-v3p1-70b-instruct"
```

---

## API Key Setup

This section provides a quick reference for setting up API keys for each provider. All API keys can be configured in three ways, listed in order of precedence:

1. **Environment variable** (highest priority) — Set via `export` or `.env` file
2. **config.yaml / config.toml** — Set the `api_key` field in the provider's section
3. **Interactive prompt** — Some providers will prompt for the key on first use

### Quick setup for all providers

```bash
# Add all your API keys to .bashrc (or .zshrc on macOS)
cat >> ~/.bashrc << 'EOF'
# ManusClaw API Keys
export OPENAI_API_KEY="sk-proj-xxx"
export ANTHROPIC_API_KEY="sk-ant-xxx"
export GOOGLE_API_KEY="AIzaSyxxx"
export MISTRAL_API_KEY="xxx"
export GROQ_API_KEY="gsk_xxx"
export AWS_ACCESS_KEY_ID="AKIAxxx"
export AWS_SECRET_ACCESS_KEY="xxx"
export AWS_REGION="us-east-1"
export HUGGINGFACE_API_KEY="hf_xxx"
export OPENROUTER_API_KEY="sk-or-xxx"
EOF

source ~/.bashrc
```

Or use the `.env` file approach:

```bash
cat > ~/.manusclaw/.env << 'EOF'
OPENAI_API_KEY=sk-proj-xxx
ANTHROPIC_API_KEY=sk-ant-xxx
GOOGLE_API_KEY=AIzaSyxxx
MISTRAL_API_KEY=xxx
GROQ_API_KEY=gsk_xxx
AWS_ACCESS_KEY_ID=AKIAxxx
AWS_SECRET_ACCESS_KEY=xxx
AWS_REGION=us-east-1
HUGGINGFACE_API_KEY=hf_xxx
OPENROUTER_API_KEY=sk-or-xxx
EOF

chmod 600 ~/.manusclaw/.env
```

---

## Credential Pool

The credential pool feature allows you to configure multiple API keys for a single provider. ManusClaw will rotate through these keys, which helps with:

- **Rate limit distribution** — Spread requests across multiple keys to avoid hitting per-key rate limits
- **Cost tracking** — Assign different keys to different projects or users
- **Redundancy** — If one key is revoked or reaches its spending limit, the next key is used automatically

### Configuring a credential pool

Add multiple keys to your `.env` file using numbered suffixes:

```env
OPENAI_API_KEY_1=sk-proj-key1xxxxxxxxxxxxxxxxxxxxxx
OPENAI_API_KEY_2=sk-proj-key2xxxxxxxxxxxxxxxxxxxxxx
OPENAI_API_KEY_3=sk-proj-key3xxxxxxxxxxxxxxxxxxxxxx

ANTHROPIC_API_KEY_1=sk-ant-key1xxxxxxxxxxxxxxxxxxxx
ANTHROPIC_API_KEY_2=sk-ant-key2xxxxxxxxxxxxxxxxxxxx
```

Or configure in `config.toml`:

```toml
[llm.openai]
api_keys = [
    "sk-proj-key1xxxxxxxxxxxxxxxxxxxxxx",
    "sk-proj-key2xxxxxxxxxxxxxxxxxxxxxx",
    "sk-proj-key3xxxxxxxxxxxxxxxxxxxxxx"
]

[llm.anthropic]
api_keys = [
    "sk-ant-key1xxxxxxxxxxxxxxxxxxxx",
    "sk-ant-key2xxxxxxxxxxxxxxxxxxxx"
]
```

When using the credential pool, ManusClaw rotates through keys using a round-robin strategy by default. If a key returns a rate limit error (HTTP 429), ManusClaw automatically switches to the next key and retries the request.

---

## Environment Variables Reference

This is a complete reference of **all** environment variables that ManusClaw v5.0.0 recognizes. Variables are organized by category.

### LLM Provider Keys

| Variable | Provider | Description | Default |
|----------|----------|-------------|---------|
| `OPENAI_API_KEY` | OpenAI | API key for OpenAI | — |
| `OPENAI_BASE_URL` | OpenAI | Custom API endpoint (e.g., Azure) | — |
| `OPENAI_ORGANIZATION` | OpenAI | Organization ID | — |
| `OPENAI_API_VERSION` | OpenAI | API version string (Azure) | — |
| `ANTHROPIC_API_KEY` | Anthropic | API key for Anthropic (Claude) | — |
| `ANTHROPIC_BASE_URL` | Anthropic | Custom API endpoint | — |
| `GOOGLE_API_KEY` | Google | API key for Google AI | — |
| `GOOGLE_CLOUD_PROJECT` | Google | GCP project ID (Vertex AI) | — |
| `GOOGLE_CLOUD_LOCATION` | Google | Vertex AI region | `us-central1` |
| `MISTRAL_API_KEY` | Mistral | API key for Mistral AI | — |
| `MISTRAL_BASE_URL` | Mistral | Custom API endpoint | — |
| `GROQ_API_KEY` | Groq | API key for Groq (fast inference) | — |
| `GROQ_BASE_URL` | Groq | Custom API endpoint | `https://api.groq.com/openai/v1` |
| `AWS_ACCESS_KEY_ID` | Bedrock | AWS access key ID | — |
| `AWS_SECRET_ACCESS_KEY` | Bedrock | AWS secret access key | — |
| `AWS_SESSION_TOKEN` | Bedrock | AWS session token (temporary creds) | — |
| `AWS_REGION` | Bedrock | AWS region | `us-east-1` |
| `OLLAMA_BASE_URL` | Ollama | Ollama server URL | `http://localhost:11434` |
| `HUGGINGFACE_API_KEY` | HuggingFace | HuggingFace API token | — |
| `OPENROUTER_API_KEY` | OpenRouter | OpenRouter API key | — |

### Search Provider Keys

| Variable | Description |
|----------|-------------|
| `GOOGLE_SEARCH_API_KEY` | Google Custom Search API key |
| `GOOGLE_SEARCH_CX` | Google Custom Search Engine ID |
| `BING_SEARCH_API_KEY` | Bing Search API key |

### ManusClaw Core Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `MANUSCLAW_CONFIG_DIR` | Configuration directory path | `~/.manusclaw` |
| `MANUSCLAW_HOME` | ManusClaw home directory | `~/.manusclaw` |
| `MANUSCLAW_WORKSPACE` | Default workspace directory path | `workspace` |
| `MANUSCLAW_SKILLS_DIR` | Directory where skills are installed | `~/.manusclaw/skills` |
| `MANUSCLAW_LOG_LEVEL` | Logging level (DEBUG, INFO, WARNING, ERROR) | `INFO` |
| `MANUSCLAW_SERVER_API_KEY` | API key for the HTTP server | — |
| `MANUSCLAW_ALLOWED_ORIGINS` | Comma-separated CORS allowed origins | `*` |
| `MANUSCLAW_PROFILE` | Active config profile name | — |
| `MANUSCLAW_MODEL_PROFILE` | Active model profile name (overrides `model_profiles.default`) | — |
| `MANUSCLAW_REDACT` | Redact sensitive data from logs (`true`/`false`) | `false` |
| `MANUSCLAW_CRON_FILE` | Path to the cron configuration file | `~/.manusclaw/cron.yaml` |
| `APP_ENV` | Application environment: `production`, `development`, `test` | `production` |

### Sandbox Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `SANDBOX_BACKEND` | Sandbox backend: `docker`, `podman`, `firejail`, `none` | `docker` |
| `SSH_SANDBOX_HOST` | SSH sandbox host address | `localhost` |
| `SSH_SANDBOX_PORT` | SSH sandbox port number | `2223` |
| `SSH_SANDBOX_IMAGE` | Docker image for SSH sandbox | `manusclaw/sandbox:latest` |

### Voice Configuration

| Variable | Description |
|----------|-------------|
| `ELEVENLABS_API_KEY` | ElevenLabs API key for TTS |
| `AZURE_TTS_KEY` | Azure Cognitive Services key for TTS |
| `AZURE_TTS_REGION` | Azure TTS region |
| `AZURE_STT_KEY` | Azure Cognitive Services key for STT |
| `AZURE_STT_REGION` | Azure STT region |

### WhatsApp Channel

| Variable | Description |
|----------|-------------|
| `WHATSAPP_ACCESS_TOKEN` | WhatsApp Business API access token |
| `WHATSAPP_PHONE_NUMBER_ID` | Phone number ID from Meta Developer Dashboard |
| `WHATSAPP_VERIFY_TOKEN` | Webhook verification token |
| `WHATSAPP_WEBHOOK_URL` | Public webhook URL for receiving messages |

### Signal Channel

| Variable | Description |
|----------|-------------|
| `SIGNAL_PHONE_NUMBER` | Signal phone number (e.g., `+15551234567`) |
| `SIGNAL_CONFIG_PATH` | Signal CLI configuration path |

### Matrix Channel

| Variable | Description |
|----------|-------------|
| `MATRIX_ACCESS_TOKEN` | Matrix access token |
| `MATRIX_HOMESERVER_URL` | Matrix homeserver URL |
| `MATRIX_USER_ID` | Matrix user ID (e.g., `@manusclaw:matrix.org`) |
| `MATRIX_DEVICE_ID` | Matrix device ID |

### IRC Channel

| Variable | Description |
|----------|-------------|
| `IRC_SERVER` | IRC server hostname |
| `IRC_PORT` | IRC server port |
| `IRC_NICKSERV_PASSWORD` | NickServ password |
| `IRC_NICKNAME` | Bot nickname |
| `IRC_CHANNELS` | Comma-separated list of channels to join |

### Twitch Channel

| Variable | Description |
|----------|-------------|
| `TWITCH_OAUTH_TOKEN` | Twitch OAuth token |
| `TWITCH_CLIENT_ID` | Twitch application client ID |
| `TWITCH_CLIENT_SECRET` | Twitch application client secret |
| `TWITCH_CHANNELS` | Comma-separated list of channels to join |

### Microsoft Teams Channel

| Variable | Description |
|----------|-------------|
| `MICROSOFT_APP_ID` | Microsoft application (client) ID |
| `MICROSOFT_APP_PASSWORD` | Microsoft application client secret |
| `MICROSOFT_TENANT_ID` | Microsoft Azure AD tenant ID |

### Google Chat Channel

| Variable | Description |
|----------|-------------|
| `GOOGLE_CHAT_PROJECT_ID` | Google Cloud project ID |
| `GOOGLE_CHAT_SUBSCRIPTION_NAME` | Pub/Sub subscription name |
| `GOOGLE_CHAT_SERVICE_ACCOUNT_JSON` | Path to service account key JSON |

### Gmail Channel

| Variable | Description |
|----------|-------------|
| `GMAIL_CREDENTIALS_PATH` | Path to OAuth 2.0 credentials JSON |
| `GMAIL_TOKEN_PATH` | Path to saved OAuth token JSON |

### SSH Server

| Variable | Description | Default |
|----------|-------------|---------|
| `MANUSCLAW_SSH_ENABLED` | Enable SSH server (`true`/`false`) | `false` |
| `MANUSCLAW_SSH_HOST` | SSH bind address | `0.0.0.0` |
| `MANUSCLAW_SSH_PORT` | SSH port number | `2222` |
| `MANUSCLAW_SSH_HOST_KEY_PATH` | Path to SSH host key file | — |
| `MANUSCLAW_SSH_AUTHORIZED_KEYS_PATH` | Path to authorized_keys file | — |
| `MANUSCLAW_SSH_USERNAME` | Required SSH username | — |

### Image Generation

| Variable | Description |
|----------|-------------|
| `FAL_KEY` | Fal.ai API key for image generation |

---

## Test Mode

ManusClaw v5.0.0 includes a built-in test mode that replaces all LLM calls with a `MockLLM` provider. This is invaluable for testing integrations, workflows, and channel configurations without consuming API credits or requiring API keys.

### Enabling test mode

Set `APP_ENV=test`:

```bash
# Via environment variable
export APP_ENV=test
manusclaw

# Or in .env
echo "APP_ENV=test" >> ~/.manusclaw/.env

# Or inline
APP_ENV=test manusclaw-server
```

### What test mode does

When `APP_ENV=test`:

1. **All LLM provider settings are ignored.** Regardless of what provider/model you configure, ManusClaw uses `MockLLM`.
2. **No API keys are required.** You can run ManusClaw without any LLM provider keys configured.
3. **Responses are deterministic.** MockLLM returns predictable, predefined responses for testing.
4. **Channel integrations are fully functional.** You can test WhatsApp, Signal, Matrix, IRC, Twitch, Teams, Google Chat, and Gmail channel connections.
5. **SSH server works normally.** The SSH server can be started and tested.
6. **Agent routing works normally.** Agent routes are evaluated correctly, and messages are dispatched to the correct agent.
7. **Configuration loading is tested.** All config files (config.yaml, config.toml, .env) are loaded and validated, so you can catch config errors.

### What test mode does NOT do

- It does not make real LLM API calls
- It does not execute real shell commands (shell_exec returns mock output)
- It does not make real web searches (web_search returns mock results)
- Tool calls are logged but return predefined mock data

### Using test mode for CI/CD

Test mode is designed for automated testing:

```yaml
# Example GitHub Actions workflow
name: Test ManusClaw Config
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install ManusClaw
        run: pip install manusclaw
      - name: Validate configuration
        run: |
          export APP_ENV=test
          export MANUSCLAW_CONFIG_DIR=./test-config
          manusclaw --validate-config
      - name: Test agent routing
        run: |
          export APP_ENV=test
          manusclaw --test-routes
      - name: Test channel connections
        run: |
          export APP_ENV=test
          manusclaw --test-channels
```

### MockLLM behavior

The MockLLM provider simulates LLM responses with configurable behavior:

| Input Type | MockLLM Response |
|-----------|-----------------|
| Chat completion | Returns a canned response: `"This is a mock response from MockLLM (test mode)."` |
| Streaming | Yields the mock response in token-sized chunks |
| Function/tool calls | Returns a predefined tool call with mock parameters |
| Embeddings | Returns a zero vector of the appropriate dimension |

### Development workflow with test mode

```bash
# 1. Start with APP_ENV=development for normal operation with logging
export APP_ENV=development
export MANUSCLAW_LOG_LEVEL=DEBUG
manusclaw

# 2. Switch to test mode to verify config without spending money
export APP_ENV=test
manusclaw --validate-config

# 3. Test specific agents
export APP_ENV=test
manusclaw --test-agent coder --input "fix the bug in auth.py"

# 4. Test channel integrations
export APP_ENV=test
manusclaw --test-channel irc --message "hello world"

# 5. Production deployment
export APP_ENV=production
unset MANUSCLAW_LOG_LEVEL  # defaults to INFO
manusclaw-server
```

### APP_ENV values

| Value | Description | LLM Behavior | Logging |
|-------|-------------|--------------|---------|
| `production` | Production mode (default) | Uses configured provider | INFO |
| `development` | Development mode | Uses configured provider | DEBUG |
| `test` | Test mode | MockLLM (no real API calls) | DEBUG |

---

## Quick Reference

### Common configuration tasks

| Task | How |
|------|-----|
| Change the active model profile | `export MANUSCLAW_MODEL_PROFILE=fast` |
| Switch config profile | `export MANUSCLAW_PROFILE=production` |
| Enable SSH server | `MANUSCLAW_SSH_ENABLED=true` or set `ssh_server.enabled: true` |
| Run in test mode | `export APP_ENV=test` |
| Change server port | Set `server.port` in config.yaml or config.toml |
| Add a new channel | Add the channel section in `config.yaml` |
| Create a new profile | `mkdir -p ~/.manusclaw/profiles/my-profile && cp ~/.manusclaw/config.yaml ~/.manusclaw/profiles/my-profile/` |
| Redact secrets from logs | `export MANUSCLAW_REDACT=true` |

### Configuration file priority (simplified)

```
Environment variables > Profile files > Global files > config.toml > Defaults
```

### Getting help

- **Validate your config:** `manusclaw --validate-config`
- **Check active config:** `manusclaw --show-config`
- **List profiles:** `manusclaw profile list`
- **Check env vars:** `manusclaw --show-env`
- **Full documentation:** [github.com/manusclaw/docs](https://github.com/manusclaw/docs)
