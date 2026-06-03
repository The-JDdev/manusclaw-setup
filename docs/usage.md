# Usage Guide — ManusClaw v4.0.0

This guide covers everything you need to know about using ManusClaw on a day-to-day basis. From launching your first session to advanced features like background tasks, memory management, and the skills system, every feature is explained with practical examples.

---

## Table of Contents

- [Starting ManusClaw](#starting-manusclaw)
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
- [Advanced Usage Patterns](#advanced-usage-patterns)

---

## Starting ManusClaw

ManusClaw provides four entry points, each serving a different purpose:

### manusclaw — The main agent

The primary way to interact with ManusClaw is through the `manusclaw` command, which launches an interactive shell where you can have a conversation with the AI agent.

```bash
# Launch the interactive shell
manusclaw

# Launch with a specific workspace
manusclaw --workspace /path/to/project

# Launch with a specific config file
manusclaw --config /path/to/config.toml

# Launch with a specific provider
manusclaw --provider anthropic --model claude-sonnet-4-20250514

# Launch in BUILD mode
manusclaw --mode BUILD
```

### manusclaw-server — HTTP API server

Run ManusClaw as a persistent HTTP server, accessible via REST API:

```bash
# Start the server on the default port (8000)
manusclaw-server

# Start with custom host and port
manusclaw-server --host 0.0.0.0 --port 9000

# Start with API key authentication
manusclaw-server --api-key your-secret-key

# Start with multiple workers
manusclaw-server --workers 4
```

### manusclaw-cron — Scheduled task runner

Run tasks on a recurring schedule:

```bash
# Start the cron daemon
manusclaw-cron

# List scheduled tasks
manusclaw-cron --list

# Add a new scheduled task
manusclaw-cron --add "0 9 * * 1" "Summarize the weekly meeting notes"
```

### manusclaw-multi — Multi-agent orchestrator

Run multiple agent instances in parallel for complex tasks:

```bash
# Start multi-agent mode with default configuration
manusclaw-multi

# Start with a specific number of agents
manusclaw-multi --agents 3

# Start with a specific task distribution strategy
manusclaw-multi --strategy round-robin
```

---

## Interactive Shell

When you launch `manusclaw` without arguments, you enter the interactive shell. This is a rich, terminal-based interface with auto-completion, syntax highlighting, and multi-line input support.

### First launch experience

```bash
$ manusclaw

╭─────────────────────────────────────────╮
│  ManusClaw v4.0.0                       │
│  Provider: openai / Model: gpt-4o       │
│  Workspace: /home/user/workspace        │
│  Mode: PLAN                             │
╰─────────────────────────────────────────╯

> Hello! How can I help you today?
```

On first launch, ManusClaw:
1. Creates the `~/.manusclaw/` configuration directory if it doesn't exist
2. Generates default `config.toml` and `.env` files
3. Initializes the workspace directory
4. Loads any existing MEMORY.md and USER.md files
5. Displays the current configuration summary

### Input modes

The interactive shell supports two input modes:

**Single-line mode (default):** Type your message and press Enter to send it. This is suitable for most interactions:

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
# Daily code review
manusclaw "Review all Python files in the workspace for potential bugs and security issues" > review-$(date +%Y%m%d).md
```

### Exit codes

ManusClaw returns meaningful exit codes in single-shot mode:

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

#### `/version` — Show version information

Displays the current ManusClaw version, Python version, and installation path.

```
> /version
ManusClaw v4.0.0
Python 3.11.9
Installation: /home/user/.local/lib/python3.11/site-packages/manusclaw
```

#### `/clear` — Clear the conversation

Clears the current conversation history, starting fresh. This is useful when the conversation context becomes too long or when you want to switch to an entirely different topic.

```
> /clear
Conversation cleared. Starting fresh.
```

**Why clear?** Long conversations consume tokens and can cause the agent to lose focus on your current task. Clearing the conversation resets the context window, giving you a clean slate.

#### `/exit` or `/quit` — Exit ManusClaw

Gracefully exits ManusClaw, saving any unsaved memory and session data.

```
> /exit
Saving memory... Done.
Goodbye!
```

### Configuration Commands

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

The `/config` command allows you to change settings on the fly without editing config.toml manually. Changes made with `/config` persist for the current session and can optionally be saved to the config file.

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

#### `/provider` — Switch LLM provider

```
# Switch to Anthropic
> /provider anthropic claude-sonnet-4-20250514

# Switch to Ollama (local)
> /provider ollama llama3

# Switch to OpenRouter
> /provider universal openai/gpt-4o
```

### Task Management Commands

#### `/bg` — Run a task in the background

The `/bg` command is one of ManusClaw's most powerful features. It allows you to start a long-running task and continue working in the foreground while the background task executes independently.

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

#### `/cancel` — Cancel the current task

```
> /cancel
Current task cancelled.
```

### Memory Commands

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

#### `/user` — View or edit user profile

```
# Show user profile
> /user

# Set a preference
> /user set preferred_language "Python"
> /user set coding_style "PEP 8 with 120 character line limit"
> /user set experience_level "senior"
```

The user profile helps ManusClaw tailor its responses to your skill level and preferences. For example, if you set your experience level to "beginner", the agent will provide more detailed explanations. If you set it to "senior", it will be more concise.

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

### Session Commands

#### `/save` — Save the current session

```
> /save
Session saved to: ~/.manusclaw/sessions/session_20241201_143022.json
```

#### `/load` — Load a previous session

```
# List saved sessions
> /load --list

# Load a specific session
> /load session_20241201_143022

# Load the most recent session
> /load --recent
```

#### `/history` — View conversation history

```
# Show recent history
> /history

# Show last 50 messages
> /history --limit 50

# Search history for a keyword
> /history --search "docker"
```

### Debug Commands

#### `/debug` — Toggle debug mode

```
> /debug on
Debug mode enabled. Detailed logging will be shown.

> /debug off
Debug mode disabled.
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
╰──────────────────────────────────────────────────╯
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
╰───────────────────────────────────────────────────╯
```

---

## Background Task Execution

The background task system is one of ManusClaw's standout features. It allows you to offload long-running tasks to background workers while you continue working interactively. This section covers background tasks in detail.

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

Sessions allow you to save and restore entire conversation states, including the message history, configuration, and context. This is useful when you work on multiple projects or want to revisit a previous conversation.

### Saving a session

```
> /save
Session saved: session_20241201_143022

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
│ api-refactor-session   │ 2024-12-01 14:30   │ 45       │ 12,340 │
│ session_20241201_0915  │ 2024-12-01 09:15   │ 23       │ 5,678  │
│ session_20241130_1630  │ 2024-11-30 16:30   │ 67       │ 28,901 │
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

This file stores information the agent learns about your project during conversations. It's automatically updated as the agent works. You can think of it as the agent's notebook.

The agent uses MEMORY.md to:
- Remember the structure of your codebase
- Track decisions made in previous sessions
- Store important patterns or conventions
- Keep notes about ongoing issues or TODOs

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
- 2024-12-01: Decided to use Celery for background tasks instead of Django-Q
- 2024-11-28: Switched from pytest to unittest for simpler test setup

## Known Issues
- The payment webhook handler has a race condition (issue #234)
- Search API is slow for queries with many filters
```

#### USER.md — User profile

This file stores information about you — your preferences, coding style, and any personal context. This helps the agent tailor its responses.

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

1. **Let the agent manage memory automatically.** The agent is good at deciding what's worth remembering. You rarely need to manually add entries.

2. **Review MEMORY.md periodically.** Check it every few sessions to make sure the information is still accurate. You can edit it directly with any text editor.

3. **Keep USER.md up to date.** If your preferences change, update USER.md. The agent reads it at the start of every session.

4. **Don't put secrets in memory files.** MEMORY.md and USER.md are plain text files. Never store API keys, passwords, or other secrets in them.

5. **Commit memory files to git.** If you're working on a team, consider committing MEMORY.md and USER.md to your project's git repository so the agent's knowledge is shared across the team.

---

## Skills System

Skills are modular extensions that add new capabilities to ManusClaw. They can be installed, enabled, and disabled without modifying the core framework. Think of them as plugins that give the agent specialized knowledge or tools.

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

ManusClaw provides a comprehensive set of built-in tools that the agent uses to accomplish tasks. Understanding these tools helps you know what the agent can do and set appropriate permission controls.

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

## Advanced Usage Patterns

### Pattern 1: Iterative development

Use ManusClaw for iterative code development where you refine code through conversation:

```
> Create a FastAPI endpoint for user registration
[Agent creates the endpoint]

> /mode BUILD
> Now add input validation using Pydantic, and write unit tests for it
[Agent modifies the code and adds tests]

> Run the tests and fix any failures
[Agent runs tests and fixes issues]

> Add type hints to all the new functions
[Agent adds type hints]
```

### Pattern 2: Code review and refactoring

Use ManusClaw to review and improve existing code:

```
> Read the file src/auth.py and identify potential security vulnerabilities
[Agent reviews the code]

> Fix the vulnerabilities you found, but keep the same API interface
[Agent applies fixes]

> Write a brief summary of the changes for the commit message
[Agent generates a commit message]
```

### Pattern 3: Multi-step workflows

Chain multiple tasks together using the task queue:

```
> /queue add "Analyze the current database schema"
> /queue add "Generate migration scripts for the new schema"
> /queue add "Create a rollback plan"
> /queue add "Document the migration process"
```

### Pattern 4: Combining foreground and background tasks

Work on one task while another runs in the background:

```
> /bg Run the full test suite and create a coverage report
Background task started (ID: task_tests)

> Meanwhile, let's work on the API documentation. Start by reading the current docs...
[You work on docs while tests run in the background]

[Background] task_tests completed. Coverage: 72%. See: workspace/coverage-report.html
```

### Pattern 5: Switching providers mid-conversation

Different providers excel at different tasks. Switch providers as needed:

```
> /provider anthropic claude-sonnet-4-20250514
> Analyze this complex algorithm and suggest optimizations
[Agent provides analysis]

> /provider openai gpt-4o
> Now implement the optimizations you suggested
[Agent implements changes]

> /provider ollama llama3
> Generate a simple README for these changes
[Agent generates documentation locally, for free]
```

### Pattern 6: Using ManusClaw as a server

Start ManusClaw as a server and interact via API:

```bash
# Start the server
manusclaw-server --api-key my-secret-key
```

Then in another terminal or from another machine:

```bash
# Send a chat message
curl -X POST http://localhost:8000/api/chat \
  -H "Authorization: Bearer my-secret-key" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "What files are in the workspace?",
    "session_id": "my-session"
  }'

# List sessions
curl -H "Authorization: Bearer my-secret-key" \
  http://localhost:8000/api/sessions

# Execute a single-shot command
curl -X POST http://localhost:8000/api/execute \
  -H "Authorization: Bearer my-secret-key" \
  -H "Content-Type: application/json" \
  -d '{
    "task": "Create a Python script that lists all CSV files in the workspace",
    "mode": "BUILD"
  }'
```

This is particularly useful for integrating ManusClaw into other applications, CI/CD pipelines, or team tools.
