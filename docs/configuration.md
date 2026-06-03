# Configuration Guide — ManusClaw v4.0.0

ManusClaw is configured through two primary files: `config.toml` for structured settings and `.env` for sensitive credentials like API keys. Understanding how these files work together is essential for getting the most out of ManusClaw. This guide explains every configuration option in detail, with examples for each LLM provider and use case.

---

## Table of Contents

- [Configuration File Locations](#configuration-file-locations)
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
- [The .env File](#the-env-file)
- [LLM Provider Configuration](#llm-provider-configuration)
  - [OpenAI](#openai)
  - [Anthropic](#anthropic)
  - [Google](#google)
  - [Mistral](#mistral)
  - [AWS Bedrock](#aws-bedrock)
  - [Ollama (Local)](#ollama-local)
  - [GGUF (Local)](#gguf-local)
  - [HuggingFace](#huggingface)
  - [Universal / OpenRouter](#universal--openrouter)
- [API Key Setup](#api-key-setup)
- [Credential Pool](#credential-pool)
- [Environment Variables Reference](#environment-variables-reference)

---

## Configuration File Locations

ManusClaw looks for configuration files in the following locations, in order of priority:

| Priority | Location | Description |
|----------|----------|-------------|
| 1 (highest) | `--config` CLI flag | Explicitly specified config path |
| 2 | `./config.toml` | Current working directory |
| 3 | `~/.manusclaw/config.toml` | User's home config directory |
| 4 | `/etc/manusclaw/config.toml` | System-wide config (Linux) |

The `.env` file follows a similar lookup pattern:

| Priority | Location |
|----------|----------|
| 1 | `--env` CLI flag |
| 2 | `./.env` |
| 3 | `~/.manusclaw/.env` |

When ManusClaw starts for the first time, it automatically creates the `~/.manusclaw/` directory with a default `config.toml` and `.env` file. You should edit these files to match your needs.

**Why two separate files?** The `config.toml` file contains structured, non-sensitive configuration (model names, provider settings, token limits) that can safely be committed to version control. The `.env` file contains secrets (API keys, tokens) that should **never** be committed to a public repository. Always add `.env` to your `.gitignore`.

---

## config.toml Reference

The `config.toml` file uses TOML format, which is a human-friendly configuration format. Here is the complete reference for all available settings, organized by section.

### Full Example config.toml

Below is a comprehensive example showing every option. You do not need to include all of these — ManusClaw uses sensible defaults for any option you omit.

```toml
# ManusClaw Configuration File
# Version: 4.0.0

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
port = 8000                        # Server port
workers = 1                        # Number of uvicorn workers
cors_origins = ["*"]              # CORS allowed origins
api_key = ""                       # API key for server authentication

[cron]
enabled = false                    # Enable the cron scheduler
timezone = "UTC"                   # Timezone for cron schedules
```

---

### LLM Configuration

The `[llm]` section controls which LLM provider and model ManusClaw uses by default. This is the most important section to configure correctly.

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

**`path`** — Can be a relative path (resolved from where you launch `manusclaw`) or an absolute path. Examples:

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

When running ManusClaw in server mode (`manusclaw-server`), these settings control the HTTP server behavior:

```toml
[server]
host = "0.0.0.0"           # Bind address (0.0.0.0 = all interfaces)
port = 8000                 # Port number
workers = 1                 # Uvicorn worker processes
cors_origins = ["*"]       # Allowed CORS origins
api_key = ""                # API key for server authentication
```

**`host`** — The network interface to bind to. Use `"0.0.0.0"` to accept connections from any IP (required for remote access), or `"127.0.0.1"` to only accept local connections (more secure).

**`api_key`** — When set, all API requests to the server must include this key in the `Authorization` header. This is critical for security when exposing the server to the internet:

```bash
# With API key
curl -H "Authorization: Bearer your-api-key" http://localhost:8000/api/chat
```

**`cors_origins`** — Controls which domains can make browser-based requests to the server. Use `["*"]` for development, but specify exact domains in production:

```toml
cors_origins = ["https://your-app.example.com", "https://admin.example.com"]
```

---

### Cron Configuration

ManusClaw includes a built-in cron scheduler for recurring tasks:

```toml
[cron]
enabled = false             # Enable/disable cron scheduling
timezone = "UTC"            # Timezone for schedule interpretation
```

Cron tasks are configured separately using the `manusclaw-cron` command. See the [Usage Guide](usage.md) for details.

---

## The .env File

The `.env` file stores environment variables, primarily API keys and other sensitive credentials. It is loaded automatically when ManusClaw starts.

### Basic .env file example

```env
# OpenAI
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Anthropic
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Google
GOOGLE_API_KEY=AIzaSyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Mistral
MISTRAL_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# AWS Bedrock
AWS_ACCESS_KEY_ID=AKIAxxxxxxxxxxxxxxxx
AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AWS_SESSION_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AWS_REGION=us-east-1

# HuggingFace
HUGGINGFACE_API_KEY=hf_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# OpenRouter / Universal
OPENROUTER_API_KEY=sk-or-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Search
GOOGLE_SEARCH_API_KEY=AIzaSyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
GOOGLE_SEARCH_CX=xxxxxxxxxxxxxxxxxxxxx
BING_SEARCH_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Server
MANUSCLAW_SERVER_API_KEY=your-secure-api-key-here
```

### How .env variables interact with config.toml

Environment variables take precedence over values in `config.toml`. This means:

1. If you set `OPENAI_API_KEY` in `.env`, it overrides `api_key` in `[llm.openai]`
2. If you set both, the environment variable wins
3. This allows you to have a shared `config.toml` while keeping secrets in `.env`

This precedence order is intentional: it lets you commit `config.toml` to version control (it contains no secrets) while keeping `.env` private.

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

5. **Don't share keys between environments.** Use different API keys for development, staging, and production.

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
base_url = "https://api.together.xyz/v1"    # Together AI
api_key = "your-together-api-key"
model = "meta-llama/Llama-3-70b-chat-hf"

# Or: Groq
base_url = "https://api.groq.com/openai/v1"
api_key = "your-groq-api-key"
model = "llama-3.1-70b-versatile"

# Or: Fireworks AI
base_url = "https://api.fireworks.ai/inference/v1"
api_key = "your-fireworks-api-key"
model = "accounts/fireworks/models/llama-v3p1-70b-instruct"
```

---

## API Key Setup

This section provides a quick reference for setting up API keys for each provider. All API keys can be configured in three ways, listed in order of precedence:

1. **Environment variable** (highest priority) — Set via `export` or `.env` file
2. **config.toml** — Set the `api_key` field in the provider's section
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

This is a complete reference of all environment variables that ManusClaw recognizes.

### LLM Provider Keys

| Variable | Provider | Description |
|----------|----------|-------------|
| `OPENAI_API_KEY` | OpenAI | API key for OpenAI |
| `OPENAI_BASE_URL` | OpenAI | Custom API endpoint |
| `OPENAI_ORGANIZATION` | OpenAI | Organization ID |
| `OPENAI_API_VERSION` | OpenAI | API version (Azure) |
| `ANTHROPIC_API_KEY` | Anthropic | API key for Anthropic |
| `ANTHROPIC_BASE_URL` | Anthropic | Custom API endpoint |
| `GOOGLE_API_KEY` | Google | API key for Google AI |
| `GOOGLE_CLOUD_PROJECT` | Google | GCP project ID (Vertex AI) |
| `GOOGLE_CLOUD_LOCATION` | Google | Vertex AI location |
| `MISTRAL_API_KEY` | Mistral | API key for Mistral |
| `MISTRAL_BASE_URL` | Mistral | Custom API endpoint |
| `AWS_ACCESS_KEY_ID` | Bedrock | AWS access key |
| `AWS_SECRET_ACCESS_KEY` | Bedrock | AWS secret key |
| `AWS_SESSION_TOKEN` | Bedrock | AWS session token |
| `AWS_REGION` | Bedrock | AWS region |
| `OLLAMA_BASE_URL` | Ollama | Ollama server URL |
| `HUGGINGFACE_API_KEY` | HuggingFace | HuggingFace API token |
| `OPENROUTER_API_KEY` | OpenRouter | OpenRouter API key |

### Search Provider Keys

| Variable | Description |
|----------|-------------|
| `GOOGLE_SEARCH_API_KEY` | Google Custom Search API key |
| `GOOGLE_SEARCH_CX` | Google Custom Search Engine ID |
| `BING_SEARCH_API_KEY` | Bing Search API key |

### ManusClaw Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `MANUSCLAW_CONFIG_DIR` | Configuration directory path | `~/.manusclaw` |
| `MANUSCLAW_WORKSPACE` | Workspace directory path | `workspace` |
| `MANUSCLAW_LOG_LEVEL` | Logging level | `INFO` |
| `MANUSCLAW_SERVER_API_KEY` | Server authentication key | (empty) |
| `MANUSCLAW_PERMISSION_MODE` | Permission mode (PLAN/BUILD) | `PLAN` |
| `MANUSCLAW_PROVIDER` | Default LLM provider | `openai` |
| `MANUSCLAW_MODEL` | Default model name | `gpt-4o` |

### Credential Pool Variables

Use numbered suffixes for multiple keys:

| Variable Pattern | Example |
|-----------------|---------|
| `OPENAI_API_KEY_1` through `OPENAI_API_KEY_N` | `OPENAI_API_KEY_1=sk-xxx` |
| `ANTHROPIC_API_KEY_1` through `ANTHROPIC_API_KEY_N` | `ANTHROPIC_API_KEY_1=sk-ant-xxx` |
| `MISTRAL_API_KEY_1` through `MISTRAL_API_KEY_N` | `MISTRAL_API_KEY_1=xxx` |

All environment variables override their corresponding `config.toml` settings. This is useful for:
- CI/CD pipelines where you inject secrets via environment variables
- Docker containers where you pass keys via `-e` flags
- Testing different configurations without modifying config files
