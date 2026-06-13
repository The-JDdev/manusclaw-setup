# Google Colab Guide — ManusClaw v5.1.0

Google Colab provides free access to GPU-equipped cloud computing environments, making it an attractive option for running ManusClaw with local models via Ollama or for users who don't have a suitable local machine. This guide walks you through setting up and using ManusClaw v5.1.0 in Colab, including how to expose the server for remote access.

---

## Table of Contents

- [Why Use Colab with ManusClaw?](#why-use-colab-with-manusclaw)
- [Colab Limitations](#colab-limitations)
- [Quick Start](#quick-start)
- [Step-by-Step Setup](#step-by-step-setup)
- [Running ManusClaw in Single-Shot Mode](#running-manusclaw-in-single-shot-mode)
- [Running the ManusClaw Server](#running-the-manusclaw-server)
- [Using Ollama on Colab (Free GPU)](#using-ollama-on-colab-free-gpu)
- [Exposing the Server via ngrok](#exposing-the-server-via-ngrok)
- [v5.1 Features in Colab](#v51-features-in-colab)
- [Managing API Keys Securely](#managing-api-keys-securely)
- [Colab Notebook Template](#colab-notebook-template)
- [Tips and Best Practices](#tips-and-best-practices)
- [Troubleshooting Colab Issues](#troubleshooting-colab-issues)

---

## Why Use Colab with ManusClaw?

Google Colab offers several advantages for ManusClaw users:

- **Free GPU access** — Colab provides T4 GPUs for free, enough to run small-to-medium LLMs via Ollama
- **No local installation needed** — Everything runs in the cloud, accessible from any device with a browser
- **Pre-configured environment** — Colab already has Python, pip, and many data science libraries installed
- **Colab Pro** — For $10/month, you get access to better GPUs (A100, V100) and longer runtimes
- **Collaboration** — Share your Colab notebook with others for collaborative AI-powered workflows

---

## Colab Limitations

Before diving in, be aware of these Colab-specific limitations:

| Limitation | Details |
|-----------|---------|
| **Runtime timeout** | Free: ~90 minutes of inactivity; Pro: ~24 hours |
| **No persistent storage** | All data is lost when the runtime disconnects |
| **No interactive terminal** | Colab cells execute code, but you can't run an interactive shell directly |
| **No audio devices** | Microphone/speakers not available — voice features **will not work** |
| **Resource limits** | RAM (12 GB free), Disk (~70 GB), GPU (T4 free, limited hours) |
| **Background execution** | The notebook must stay open; closing the tab stops execution |
| **Network restrictions** | Some ports and protocols may be blocked |

The biggest challenge is the lack of an interactive terminal. ManusClaw's primary interface is a REPL (Read-Eval-Print Loop), which doesn't work natively in Colab. We'll work around this using single-shot mode, the server API, and ngrok tunneling.

### Feature Availability in Colab

| Feature | Available | Notes |
|---------|-----------|-------|
| Single-shot mode | ✅ | Primary usage method |
| Server mode (API) | ✅ | Access via HTTP/WebSocket |
| Ollama (local LLMs) | ✅ | With GPU runtime |
| Channels (Telegram, Discord, etc.) | ✅ | Via server API |
| Webhooks | ✅ | Requires ngrok |
| Canvas (WebChat) | ✅ | Via WebSocket |
| Cron scheduling | ⚠️ | Limited by runtime timeout |
| Voice (wake/talk) | ❌ | No audio devices |
| SSH Gateway | ❌ | No terminal access |
| Security / RBAC | ✅ | Server-side only |
| Hooks | ✅ | Server-side only |
| Context management | ✅ | Automatic |
| Observability | ⚠️ | No Prometheus; structured logging only |
| Secrets (Vault) | ⚠️ | Can connect to external Vault |
| File store (S3) | ✅ | Use S3 backend for persistence |
| Git providers | ✅ | API-based access |
| Parallel executor | ✅ | Threaded mode only |
| Migrations | ✅ | Config migrations |

---

## Quick Start

For the fastest possible setup, create a new Colab notebook and paste this into a code cell:

```python
# Cell 1: Install ManusClaw v5.1 with all extras
!pip install "manusclaw[all]"

# Cell 2: Set API key and run
import os
os.environ['OPENAI_API_KEY'] = 'sk-proj-your-key-here'
!manusclaw "What can you do?"
```

This works for quick, one-off queries. For more sophisticated usage, follow the detailed setup below.

---

## Step-by-Step Setup

### Step 1: Create a new Colab notebook

1. Go to [colab.research.google.com](https://colab.research.google.com/)
2. Click "New notebook" (or File → New notebook)
3. Rename it to something like "ManusClaw v5.1 Setup"

### Step 2: Change runtime type (for GPU support)

If you want to use Ollama with a GPU:

1. Click Runtime → Change runtime type
2. Under "Hardware accelerator", select **T4 GPU**
3. Click Save

Verify GPU access:

```python
!nvidia-smi
```

You should see GPU information. If you see "NVIDIA-SMI has failed," the GPU isn't available — try again later or use Colab Pro.

### Step 3: Install ManusClaw

```python
# Install ManusClaw v5.1 with all optional dependencies
!pip install "manusclaw[all]"
```

### Step 4: Configure API keys

**Option A: Direct environment variable (simple but visible in notebook)**

```python
import os
os.environ['OPENAI_API_KEY'] = 'sk-proj-your-key-here'
```

**Option B: Use Colab Secrets (recommended for shared notebooks)**

1. Click the 🔑 key icon in the left sidebar
2. Click "Add a new secret"
3. Name: `OPENAI_API_KEY`, Value: your API key
4. Toggle "Notebook access" to on

Then in a code cell:

```python
from google.colab import userdata
import os

os.environ['OPENAI_API_KEY'] = userdata.get('OPENAI_API_KEY')
```

**Option C: Upload a .env file**

```python
from google.colab import files
import shutil

uploaded = files.upload()
env_filename = list(uploaded.keys())[0]

!mkdir -p ~/.manusclaw
shutil.move(env_filename, '/root/.manusclaw/.env')
```

### Step 5: Create workspace directory

```python
!mkdir -p workspace
```

---

## Running ManusClaw in Single-Shot Mode

Single-shot mode is the simplest way to use ManusClaw in Colab. You provide a prompt, and ManusClaw responds once:

```python
# Basic usage
!manusclaw "Explain the difference between a list and a tuple in Python"
```

### Using Python variables in prompts

```python
filename = "data.csv"
prompt = f"Analyze the file {filename} and describe its structure"
!manusclaw "{prompt}"
```

### Piping data into ManusClaw

```python
# Analyze a file
!manusclaw "Summarize the key points" < README.md

# Process command output
!echo "Error: connection refused at port 8080" | manusclaw "What does this error mean?"
```

### Using Python to call ManusClaw

```python
import subprocess

def ask_manusclaw(prompt, provider="openai", model="gpt-4o-mini"):
    """Send a single-shot query to ManusClaw and return the response."""
    result = subprocess.run(
        ["manusclaw", "--provider", provider, "--model", model, prompt],
        capture_output=True,
        text=True,
        timeout=120
    )
    if result.returncode == 0:
        return result.stdout
    else:
        return f"Error: {result.stderr}"

response = ask_manusclaw("Write a Python function to calculate Fibonacci numbers")
print(response)
```

### Capturing output to a file

```python
!manusclaw "Create a Python Flask REST API with CRUD endpoints" > api_code.py
!cat api_code.py
```

---

## Running the ManusClaw Server

Running ManusClaw in server mode allows you to interact with it via HTTP API, which is more flexible than single-shot mode.

### Start the server in the background

```python
import subprocess
import time

server_process = subprocess.Popen(
    ["manusclaw-server", "--host", "0.0.0.0", "--port", "8765"],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE
)

time.sleep(5)

import requests
try:
    response = requests.get("http://localhost:8765/health")
    print(f"Server status: {response.json()}")
except Exception as e:
    print(f"Server not ready: {e}")
```

### Send messages to the server

```python
import requests

def chat(message, session_id="colab-session"):
    """Send a chat message to the ManusClaw server."""
    url = "http://localhost:8765/api/chat"
    payload = {
        "message": message,
        "session_id": session_id
    }
    response = requests.post(url, json=payload, timeout=300)
    return response.json()

result = chat("What is the capital of France?")
print(result.get("response", result))
```

### Interactive chat interface

```python
from IPython.display import display, HTML, clear_output
import ipywidgets as widgets

output = widgets.Output()
text_input = widgets.Text(
    value='',
    placeholder='Type your message...',
    description='You:',
    layout=widgets.Layout(width='80%')
)

def on_submit(change):
    message = change.value
    text_input.value = ''
    with output:
        print(f"\nYou: {message}")
        try:
            result = chat(message)
            print(f"ManusClaw: {result.get('response', result)}")
        except Exception as e:
            print(f"Error: {e}")

text_input.on_submit(on_submit)
display(widgets.VBox([output, text_input]))
```

---

## Using Ollama on Colab (Free GPU)

One of the biggest advantages of Colab is free GPU access, which enables running local LLMs via Ollama. This means you can use ManusClaw for free, without any API keys.

### Step 1: Install Ollama

```python
!curl -fsSL https://ollama.com/install.sh | sh
```

### Step 2: Start Ollama in the background

```python
import subprocess, time

ollama_process = subprocess.Popen(
    ["ollama", "serve"],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE
)
time.sleep(5)
!curl http://localhost:11434/api/tags
```

### Step 3: Pull a model

```python
# Small models for T4 GPU:
!ollama pull phi3          # 2.3 GB - very fast, smaller
!ollama pull mistral       # 4.1 GB - fast
!ollama pull llama3        # 4.7 GB - good balance
!ollama list
```

### Step 4: Configure ManusClaw to use Ollama

```python
import os
os.environ['MANUSCLAW_PROVIDER'] = 'ollama'
os.environ['MANUSCLAW_MODEL'] = 'llama3'
os.environ['OLLAMA_BASE_URL'] = 'http://localhost:11434'
```

### GPU model selection guide

| Model | Size | RAM Needed | T4 GPU | Quality | Speed |
|-------|------|-----------|--------|---------|-------|
| phi3:mini | 2.3 GB | 4 GB | ✅ Fast | Good | ⚡⚡⚡ |
| mistral:7b | 4.1 GB | 8 GB | ✅ Good | Very Good | ⚡⚡ |
| llama3:8b | 4.7 GB | 8 GB | ✅ Good | Very Good | ⚡⚡ |
| codellama:13b | 7.4 GB | 16 GB | ⚠️ Slow | Excellent | ⚡ |
| llama3:70b | 40 GB | 64 GB | ❌ No | Best | — |

---

## Exposing the Server via ngrok

ngrok creates a secure tunnel from the public internet to your Colab runtime, allowing you to access the ManusClaw server from any device.

### Step 1: Install ngrok

```python
!pip install pyngrok
```

### Step 2: Set up ngrok authentication

```python
from pyngrok import ngrok
from google.colab import userdata

ngrok.set_auth_token(userdata.get('NGROK_AUTH_TOKEN'))
```

### Step 3: Start the tunnel

```python
from pyngrok import ngrok
import subprocess, time

# Start ManusClaw server
server_process = subprocess.Popen(
    ["manusclaw-server", "--host", "0.0.0.0", "--port", "8765"],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE
)
time.sleep(5)

# Create the ngrok tunnel (v5 server port is 8765)
public_url = ngrok.connect(8765)
print(f"ManusClaw server accessible at: {public_url}")
```

### Step 4: Access from anywhere

```bash
curl -X POST https://xxxx-xx-xx-xxx-xx.ngrok-free.app/api/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello, ManusClaw!"}'
```

### Important security note

The ngrok URL is publicly accessible. Always set a server API key:

```python
import os
os.environ['MANUSCLAW_SERVER_API_KEY'] = 'your-secure-random-key'
```

---

## v5.1 Features in Colab

### Context Management

Context management works automatically in Colab. Configure it via the server API:

```python
import requests

# Check context status
r = requests.get("http://localhost:8765/api/context/status")
print(r.json())

# Trigger manual compression
r = requests.post("http://localhost:8765/api/context/compress")
print(r.json())
```

### File Store with S3 (Persistent Storage)

Use S3 as the file store backend for persistence across Colab sessions:

```python
import os

# Configure S3 file store
os.environ['AWS_ACCESS_KEY_ID'] = 'your-aws-key'
os.environ['AWS_SECRET_ACCESS_KEY'] = 'your-aws-secret'
os.environ['AWS_DEFAULT_REGION'] = 'us-east-1'
```

Then in `config.yaml`:

```yaml
file_store:
  backend: "s3"
  s3:
    bucket: "manusclaw-colab-artifacts"
    prefix: "colab/"
```

### Git Providers

Use Git providers to interact with repositories directly from Colab:

```python
import os
os.environ['GITHUB_TOKEN'] = 'your-github-token'

# Agent can now read repos, create issues, etc.
!manusclaw "List open issues in my-org/my-repo and summarize them"
```

### Parallel Executor

Use threaded parallel execution in Colab:

```python
import requests

# Run multiple tasks in parallel
r = requests.post("http://localhost:8765/api/parallel/run", json={
    "tasks": [
        {"prompt": "Analyze the sales data"},
        {"prompt": "Generate a marketing report"},
        {"prompt": "Check for anomalies"}
    ]
})
print(r.json())
```

---

## Managing API Keys Securely

### Using Colab Secrets (recommended)

```python
from google.colab import userdata
import os

for key in ['OPENAI_API_KEY', 'ANTHROPIC_API_KEY', 'GOOGLE_API_KEY',
            'GITHUB_TOKEN', 'NGROK_AUTH_TOKEN']:
    try:
        os.environ[key] = userdata.get(key)
        print(f"✓ {key} set")
    except:
        print(f"✗ {key} not found in secrets")
```

### Using Google Drive for persistent config

```python
from google.colab import drive
drive.mount('/content/drive')

!mkdir -p /content/drive/MyDrive/manusclaw
!ln -sf /content/drive/MyDrive/manusclaw ~/.manusclaw
```

---

## Colab Notebook Template

```python
# ============================================================
# Cell 1: Installation (v5.1 with all extras)
# ============================================================
!pip install "manusclaw[all]"

# ============================================================
# Cell 2: Configuration
# ============================================================
import os
from google.colab import userdata

for key in ['OPENAI_API_KEY', 'ANTHROPIC_API_KEY', 'GOOGLE_API_KEY',
            'GITHUB_TOKEN', 'MANUSCLAW_API_KEY']:
    try:
        os.environ[key] = userdata.get(key)
        print(f"✓ {key} set")
    except:
        print(f"✗ {key} not found in secrets")

!mkdir -p workspace

# ============================================================
# Cell 3: Option A - Single-shot mode
# ============================================================
!manusclaw "Hello! What can you help me with?"

# ============================================================
# Cell 4: Option B - Server mode with ngrok
# ============================================================
import subprocess, time
from pyngrok import ngrok

server = subprocess.Popen(
    ["manusclaw-server", "--host", "0.0.0.0", "--port", "8765"],
    stdout=subprocess.PIPE, stderr=subprocess.PIPE
)
time.sleep(5)

try:
    ngrok.set_auth_token(userdata.get('NGROK_AUTH_TOKEN'))
    public_url = ngrok.connect(8765)
    print(f"🌐 ManusClaw server: {public_url}")
except:
    print("⚠️ Set NGROK_AUTH_TOKEN in secrets for remote access")
    print("Server running locally at http://localhost:8765")

# ============================================================
# Cell 5: Chat with ManusClaw
# ============================================================
import requests

def chat(message):
    try:
        r = requests.post(
            "http://localhost:8765/api/chat",
            json={"message": message, "session_id": "colab"},
            timeout=300
        )
        return r.json().get("response", r.text)
    except Exception as e:
        return f"Error: {e}"

response = chat("Explain what ManusClaw v5.1 can do")
print(response)

# ============================================================
# Cell 6: Option C - Ollama (Free, no API keys needed)
# ============================================================
!curl -fsSL https://ollama.com/install.sh | sh
import subprocess, time
ollama = subprocess.Popen(["ollama", "serve"],
    stdout=subprocess.PIPE, stderr=subprocess.PIPE)
time.sleep(5)
!ollama pull phi3
os.environ['MANUSCLAW_PROVIDER'] = 'ollama'
os.environ['MANUSCLAW_MODEL'] = 'phi3'
!manusclaw "Hello from Colab with Ollama!"
```

---

## Tips and Best Practices

### 1. Keep your runtime alive

```python
import time
from google.colab import output

while True:
    time.sleep(60)
    output.eval_js('document.title = "ManusClaw - Active"')
    print(".", end="", flush=True)
```

> **Warning:** This is against Google's terms of service for free Colab. Use responsibly or upgrade to Colab Pro.

### 2. Save your work frequently

```python
from google.colab import drive
drive.mount('/content/drive')
!cp workspace/output.txt /content/drive/MyDrive/
```

### 3. Use GPU efficiently

```python
!nvidia-smi
```

### 4. Clean up when done

```python
server.terminate()
ngrok.kill()

import torch
torch.cuda.empty_cache()
```

---

## Troubleshooting Colab Issues

### Error: `manusclaw: command not found`

```python
!pip install manusclaw --force-reinstall
!which manusclaw
!/usr/local/bin/manusclaw --version
```

### Error: Ollama model download is too slow

```python
!df -h
!ollama pull phi3  # Smaller model
```

### Error: `CUDA out of memory`

```python
!OLLAMA_NUM_GPU=0 ollama run llama3  # Force CPU mode
import torch
torch.cuda.empty_cache()
```

### Error: ngrok tunnel limit

```python
ngrok.kill()
public_url = ngrok.connect(8765)
```

### Error: Runtime disconnected

Colab's free tier disconnects after 90 minutes of inactivity. To preserve your work:

1. Save outputs to Google Drive
2. Use version control (git) for any code changes
3. Consider Colab Pro for longer sessions
