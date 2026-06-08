# Google Colab Guide — ManusClaw v5.0.0

Google Colab provides free access to GPU-equipped cloud computing environments, making it an attractive option for running ManusClaw with local models via Ollama or for users who don't have a suitable local machine. This guide walks you through setting up and using ManusClaw in Colab, including how to expose the server for remote access.

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
- [Managing API Keys Securely](#managing-api-keys-securely)
- [Colab Notebook Template](#colab-notebook-template)
- [Tips and Best Practices](#tips-and-best-practices)
- [Troubleshooting Colab Issues](#troubleshooting-colab-issues)

---

## Why Use Colab with ManusClaw?

Google Colab offers several advantages for ManusClaw users:

- **Free GPU access** — Colab provides T4 GPUs for free, which is enough to run small-to-medium LLMs via Ollama
- **No local installation needed** — Everything runs in the cloud, so you can use ManusClaw from any device with a browser
- **Pre-configured environment** — Colab already has Python, pip, and many data science libraries installed
- **Colab Pro** — For $10/month, you get access to better GPUs (A100, V100) and longer runtimes
- **Collaboration** — You can share your Colab notebook with others, making it easy to collaborate on AI-powered workflows

---

## Colab Limitations

Before diving in, be aware of these Colab-specific limitations:

| Limitation | Details |
|-----------|---------|
| **Runtime timeout** | Free: ~90 minutes of inactivity; Pro: ~24 hours |
| **No persistent storage** | All data is lost when the runtime disconnects |
| **No interactive terminal** | Colab cells execute code, but you can't run an interactive shell directly |
| **No audio devices** | Microphone/speakers are not available — v5 voice features (wake word, talk mode) **will not work** |
| **Resource limits** | RAM (12 GB free), Disk (~70 GB), GPU (T4 free, limited hours) |
| **Background execution** | The notebook must stay open; closing the tab stops execution |
| **Network restrictions** | Some ports and protocols may be blocked |

The biggest challenge is the lack of an interactive terminal. ManusClaw's primary interface is a REPL (Read-Eval-Print Loop), which doesn't work natively in Colab. We'll work around this using single-shot mode, the server API, and ngrok tunneling.

### v5 Voice Feature Limitations

ManusClaw v5.0.0 includes voice features (wake word detection, talk mode) that require audio hardware. **These features are NOT available in Google Colab** because:

- No microphone access for speech-to-text
- No speaker access for text-to-speech
- No PortAudio support in the Colab environment

All other v5 features work normally: channels, webhooks, SSH, Gmail, multi-agent routing, model failover, canvas (WebChat), session tools, enhanced cron, and all 10+ LLM providers.

---

## Quick Start

For the fastest possible setup, create a new Colab notebook and paste this into a code cell:

```python
# Cell 1: Install ManusClaw (v5 with all extras)
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
3. Rename it to something like "ManusClaw Setup"

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

In a new code cell:

```python
# Install ManusClaw v5 with all optional dependencies
!pip install "manusclaw[all]"
```

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

# Upload .env file from your computer
uploaded = files.upload()
env_filename = list(uploaded.keys())[0]

# Move it to the ManusClaw config directory
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

You can embed Python variables in your prompts using f-strings:

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
!echo "Error: connection refused at port 8080" | manusclaw "What does this error mean and how do I fix it?"
```

### Using Python to call ManusClaw

For more control, use Python's subprocess module:

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

# Usage
response = ask_manusclaw("Write a Python function to calculate Fibonacci numbers")
print(response)
```

### Capturing output to a file

```python
# Save ManusClaw's output to a file
!manusclaw "Create a Python Flask REST API with CRUD endpoints" > api_code.py

# Display the result
!cat api_code.py
```

---

## Running the ManusClaw Server

Running ManusClaw in server mode allows you to interact with it via HTTP API, which is more flexible than single-shot mode.

### Start the server in the background

```python
import subprocess
import time

# Start ManusClaw server in the background
server_process = subprocess.Popen(
    ["manusclaw-server", "--host", "0.0.0.0", "--port", "8000"],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE
)

# Wait for the server to start
time.sleep(5)

# Check if the server is running
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
import json

def chat(message, session_id="colab-session"):
    """Send a chat message to the ManusClaw server."""
    url = "http://localhost:8765/api/chat"
    payload = {
        "message": message,
        "session_id": session_id
    }
    headers = {"Content-Type": "application/json"}
    
    response = requests.post(url, json=payload, headers=headers, timeout=300)
    return response.json()

# Usage
result = chat("What is the capital of France?")
print(result.get("response", result))
```

### Interactive chat loop

Create a simple interactive chat interface within Colab:

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
import subprocess

# Start Ollama server
ollama_process = subprocess.Popen(
    ["ollama", "serve"],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE
)

# Wait for Ollama to start
import time
time.sleep(5)

# Verify Ollama is running
!curl http://localhost:11434/api/tags
```

### Step 3: Pull a model

```python
# Pull a model (this downloads it to the Colab runtime)
# Small models for T4 GPU:
!ollama pull llama3        # 4.7 GB - good balance
!ollama pull mistral       # 4.1 GB - fast
!ollama pull phi3          # 2.3 GB - very fast, smaller

# Verify the model is available
!ollama list
```

### Step 4: Configure ManusClaw to use Ollama

```python
import os
os.environ['MANUSCLAW_PROVIDER'] = 'ollama'
os.environ['MANUSCLAW_MODEL'] = 'llama3'
os.environ['OLLAMA_BASE_URL'] = 'http://localhost:11434'
```

Or create a config file:

```python
!mkdir -p ~/.manusclaw
with open('/root/.manusclaw/config.toml', 'w') as f:
    f.write("""[llm]
provider = "ollama"
model = "llama3"

[llm.ollama]
base_url = "http://localhost:11434"
model = "llama3"
timeout = 300
""")
```

### Step 5: Use ManusClaw with the local model

```python
!manusclaw "Write a haiku about programming"
```

### GPU model selection guide for Colab

| Model | Size | RAM Needed | T4 GPU | Quality | Speed |
|-------|------|-----------|--------|---------|-------|
| phi3:mini | 2.3 GB | 4 GB | ✅ Fast | Good | ⚡⚡⚡ |
| mistral:7b | 4.1 GB | 8 GB | ✅ Good | Very Good | ⚡⚡ |
| llama3:8b | 4.7 GB | 8 GB | ✅ Good | Very Good | ⚡⚡ |
| codellama:13b | 7.4 GB | 16 GB | ⚠️ Slow | Excellent | ⚡ |
| llama3:70b | 40 GB | 64 GB | ❌ No | Best | — |

The T4 GPU has 16 GB of VRAM, which can comfortably run 7B-8B models and struggle with 13B+ models. For larger models, use Colab Pro with an A100 GPU.

---

## Exposing the Server via ngrok

ngrok creates a secure tunnel from the public internet to your Colab runtime, allowing you to access the ManusClaw server from any device.

### Step 1: Install ngrok

```python
!pip install pyngrok
```

### Step 2: Set up ngrok authentication

1. Sign up at [ngrok.com](https://ngrok.com/) (free)
2. Get your authtoken from the dashboard

```python
from pyngrok import ngrok

# Set your ngrok authtoken
ngrok.set_auth_token("your-ngrok-authtoken")
```

Or use Colab Secrets:

```python
from google.colab import userdata
ngrok.set_auth_token(userdata.get('NGROK_AUTH_TOKEN'))
```

### Step 3: Start the tunnel

```python
from pyngrok import ngrok

# Start ManusClaw server (if not already running)
import subprocess
server_process = subprocess.Popen(
    ["manusclaw-server", "--host", "0.0.0.0", "--port", "8000"],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE
)

import time
time.sleep(5)

# Create the ngrok tunnel
public_url = ngrok.connect(8000)
print(f"ManusClaw server accessible at: {public_url}")
```

### Step 4: Access from anywhere

Use the ngrok URL to access ManusClaw from any device:

```bash
# From your local machine or any other device:
curl -X POST https://xxxx-xx-xx-xxx-xx.ngrok-free.app/api/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello, ManusClaw!"}'
```

### Important security note

The ngrok URL is publicly accessible. Anyone with the URL can send requests to your ManusClaw server. Always set a server API key:

```python
import os
os.environ['MANUSCLAW_SERVER_API_KEY'] = 'your-secure-random-key'

# Then include the key in requests:
# curl -H "Authorization: Bearer your-secure-random-key" ...

### ngrok for v5 server mode (port 8765)

The v5 default server port is **8765** (changed from 8000). Make sure ngrok tunnels the correct port:

```python
# In Step 3, the tunnel connects to port 8765:
public_url = ngrok.connect(8765)
print(f"🌐 ManusClaw v5 server: {public_url}")
```

### Disconnect ngrok

```python
ngrok.disconnect(public_url)
# Or kill all tunnels:
ngrok.kill()
```

---

## Managing API Keys Securely

### Using Colab Secrets (recommended)

Colab's built-in secret management is the most secure way to store API keys in notebooks:

```python
from google.colab import userdata
import os

# Set up all your API keys
try:
    os.environ['OPENAI_API_KEY'] = userdata.get('OPENAI_API_KEY')
except:
    print("OPENAI_API_KEY not set in Colab Secrets")

try:
    os.environ['ANTHROPIC_API_KEY'] = userdata.get('ANTHROPIC_API_KEY')
except:
    print("ANTHROPIC_API_KEY not set in Colab Secrets")

try:
    os.environ['GOOGLE_API_KEY'] = userdata.get('GOOGLE_API_KEY')
except:
    print("GOOGLE_API_KEY not set in Colab Secrets")
```

### Using Google Drive for persistent config

To persist your configuration across Colab sessions:

```python
from google.colab import drive
drive.mount('/content/drive')

# Create a ManusClaw config directory on Drive
!mkdir -p /content/drive/MyDrive/manusclaw

# Create a symlink so ManusClaw finds the config
!ln -sf /content/drive/MyDrive/manusclaw ~/.manusclaw

# Now your config persists between sessions
```

---

## Colab Notebook Template

Here's a complete Colab notebook template that you can copy and paste into a new notebook:

```python
# ============================================================
# Cell 1: Installation (v5 with all extras)
# ============================================================
!pip install "manusclaw[all]"

# ============================================================
# Cell 2: Configuration
# ============================================================
import os
from google.colab import userdata

# Set API keys from Colab Secrets
# Add your keys in the 🔑 sidebar before running this cell
for key in ['OPENAI_API_KEY', 'ANTHROPIC_API_KEY', 'GOOGLE_API_KEY', 'MANUSCLAW_API_KEY']:
    try:
        os.environ[key] = userdata.get(key)
        print(f"✓ {key} set")
    except:
        print(f"✗ {key} not found in secrets")

# Create workspace
!mkdir -p workspace

# ============================================================
# Cell 3: Option A - Single-shot mode
# ============================================================
!manusclaw "Hello! What can you help me with?"

# ============================================================
# Cell 4: Option B - Server mode with ngrok
# ============================================================
import subprocess
import time
from pyngrok import ngrok

# Start ManusClaw server
server = subprocess.Popen(
    ["manusclaw-server", "--host", "0.0.0.0", "--port", "8000"],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE
)
time.sleep(5)

# Create ngrok tunnel
try:
    ngrok.set_auth_token(userdata.get('NGROK_AUTH_TOKEN'))
    public_url = ngrok.connect(8000)
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

response = chat("Explain what ManusClaw can do")
print(response)

# ============================================================
# Cell 6: Option C - Ollama (Free, no API keys needed)
# ============================================================
# Install and start Ollama
!curl -fsSL https://ollama.com/install.sh | sh
import subprocess, time
ollama = subprocess.Popen(["ollama", "serve"],
    stdout=subprocess.PIPE, stderr=subprocess.PIPE)
time.sleep(5)

# Pull a small model
!ollama pull phi3

# Configure ManusClaw for Ollama
os.environ['MANUSCLAW_PROVIDER'] = 'ollama'
os.environ['MANUSCLAW_MODEL'] = 'phi3'

# Test
!manusclaw "Hello from Colab!"
```

---

## Tips and Best Practices

### 1. Keep your runtime alive

Colab disconnects after ~90 minutes of inactivity. To prevent this, add a cell that periodically outputs:

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

Colab data is ephemeral. Save important outputs to Google Drive:

```python
from google.colab import drive
drive.mount('/content/drive')

# Save ManusClaw output
!cp workspace/output.txt /content/drive/MyDrive/
```

### 3. Use GPU efficiently

If you're using Ollama, monitor GPU usage:

```python
!nvidia-smi
```

### 4. Clean up when done

```python
# Stop the server
server.terminate()
ngrok.kill()

# Free up GPU memory
import torch
torch.cuda.empty_cache()
```

---

## Troubleshooting Colab Issues

### Error: `manusclaw: command not found`

```python
# ManusClaw wasn't installed properly — reinstall
!pip install manusclaw --force-reinstall

# Check if it's in the PATH
!which manusclaw

# If not, use the full path
!/usr/local/bin/manusclaw --version
```

### Error: Ollama model download is too slow

```python
# Check available disk space
!df -h

# Use a smaller model
!ollama pull phi3  # 2.3 GB instead of llama3's 4.7 GB
```

### Error: `CUDA out of memory`

```python
# Reduce model size or use CPU mode
!OLLAMA_NUM_GPU=0 ollama run llama3  # Force CPU mode

# Or clear GPU cache
import torch
torch.cuda.empty_cache()
```

### Error: ngrok tunnel limit

Free ngrok accounts are limited to 1 tunnel. If you get an error about too many tunnels:

```python
# Kill existing tunnels
ngrok.kill()

# Then create a new one
public_url = ngrok.connect(8765)
```

### Error: Runtime disconnected

Colab's free tier disconnects after 90 minutes of inactivity or 12 hours of continuous use. There's no workaround for the hard time limit. To preserve your work:

1. Save outputs to Google Drive
2. Use version control (git) for any code changes
3. Consider Colab Pro for longer sessions
