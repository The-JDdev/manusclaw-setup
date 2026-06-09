# Deployment Guide — ManusClaw v5.0.0

This guide covers deploying ManusClaw in production environments. Whether you're running it on a VPS, in Docker, or behind a reverse proxy, this guide provides battle-tested configurations for reliable, secure, and maintainable deployments.

---

## Table of Contents

- [Deployment Overview](#deployment-overview)
- [Docker Deployment](#docker-deployment)
- [docker-compose Reference](#docker-compose-reference)
- [VPS Deployment](#vps-deployment)
- [Systemd Service Setup](#systemd-service-setup)
- [Nginx Reverse Proxy](#nginx-reverse-proxy)
- [SSL/TLS Setup](#ssltls-setup)
- [Auto-Start on Boot](#auto-start-on-boot)
- [Background Execution](#background-execution)
- [Process Management with Supervisord](#process-management-with-supervisord)
- [Channel Adapter Deployment](#channel-adapter-deployment)
- [Security Recommendations](#security-recommendations)
- [Resource Requirements](#resource-requirements)
- [Scaling Considerations](#scaling-considerations)

---

## Deployment Overview

ManusClaw can be deployed in several ways depending on your needs:

| Method | Best For | Complexity | Scalability |
|--------|----------|-----------|-------------|
| **Docker** | Isolated, reproducible deployments | Low | Medium |
| **VPS (bare metal)** | Maximum control, custom tuning | Medium | Low-Medium |
| **Docker Compose** | Multi-service deployments with profiles | Medium | Medium |
| **Kubernetes** | Large-scale, auto-scaling | High | High |

For most users and small teams, Docker or a simple VPS deployment is the right choice. Kubernetes is only necessary if you need to handle very high traffic or require automatic scaling.

### v5 Deployment Modes

ManusClaw v5.0.0 introduces three deployment profiles:

| Profile | Components | Use Case |
|---------|-----------|----------|
| **server** | HTTP server + WebSocket + WebChat + Webhooks | API access, web UI, webhook ingestion |
| **cli** | Interactive REPL + session tools | Single-user local or SSH usage |
| **multi-agent** | Server + Channels + SSH gateway + Cron | Full multi-channel AI agent platform |

---

## Docker Deployment

Docker is the recommended deployment method because it ensures a consistent, isolated environment regardless of the host OS. The ManusClaw repository includes a production-ready Dockerfile.

### Building the Docker image

```bash
# Clone the repository
git clone https://github.com/ManusClawAI/manusclaw.git
cd manusclaw

# Build the image
docker build -t manusclaw:5.0.0 .

# Tag as latest
docker tag manusclaw:5.0.0 manusclaw:latest
```

### Running the container

#### Basic server mode (v5 default port 8765)

```bash
docker run -d \
  --name manusclaw-server \
  --restart unless-stopped \
  -p 8765:8765 \
  -e OPENAI_API_KEY="sk-your-key-here" \
  -e MANUSCLAW_API_KEY="your-secure-key" \
  -v manusclaw-config:/root/.manusclaw \
  -v manusclaw-workspace:/app/workspace \
  manusclaw:latest \
  manusclaw-server --host 0.0.0.0 --port 8765
```

Let's break down each flag:

- **`-d`** — Run in detached mode (background)
- **`--name manusclaw-server`** — Give the container a recognizable name
- **`--restart unless-stopped`** — Automatically restart if it crashes, but not if you manually stop it
- **`-p 8765:8765`** — Map host port 8765 to container port 8765
- **`-e OPENAI_API_KEY=...`** — Pass API keys as environment variables
- **`-e MANUSCLAW_API_KEY=...`** — Set a server authentication key
- **`-v manusclaw-config:/root/.manusclaw`** — Persist config to a named volume
- **`-v manusclaw-workspace:/app/workspace`** — Persist workspace to a named volume

#### Interactive shell mode

```bash
docker run -it --rm \
  -e OPENAI_API_KEY="sk-your-key-here" \
  -v $(pwd)/workspace:/app/workspace \
  -v $(pwd)/config:/root/.manusclaw \
  manusclaw:latest \
  manusclaw
```

#### With Ollama integration

If you want to use a local Ollama instance alongside ManusClaw, you need to connect the containers:

```bash
# Create a Docker network
docker network create manusclaw-net

# Run Ollama
docker run -d \
  --name ollama \
  --restart unless-stopped \
  --gpus all \
  -v ollama-data:/root/.ollama \
  --network manusclaw-net \
  ollama/ollama:latest

# Pull a model (first time only)
docker exec ollama ollama pull llama3

# Run ManusClaw connected to Ollama
docker run -d \
  --name manusclaw-server \
  --restart unless-stopped \
  -p 8765:8765 \
  -e OLLAMA_BASE_URL="http://ollama:11434" \
  -e MANUSCLAW_PROVIDER="ollama" \
  -e MANUSCLAW_MODEL="llama3" \
  --network manusclaw-net \
  manusclaw:latest \
  manusclaw-server --host 0.0.0.0
```

### Managing Docker containers

```bash
# View running containers
docker ps

# View logs
docker logs manusclaw-server

# Follow logs in real-time
docker logs -f manusclaw-server

# Restart the container
docker restart manusclaw-server

# Stop the container
docker stop manusclaw-server

# Remove the container
docker rm manusclaw-server

# Execute a command inside the container
docker exec -it manusclaw-server /bin/bash

# Update to a new version
docker pull manusclaw:latest
docker stop manusclaw-server
docker rm manusclaw-server
# Re-run with the same docker run command
```

---

## docker-compose Reference

The `docker-compose.yml` file provides a more manageable way to configure and run ManusClaw, especially when you have multiple services (ManusClaw, Ollama, databases, etc.).

> **v5 Note:** The `version:` field is deprecated in modern Docker Compose (v2+). Compose files now use **profiles** instead of the `version: "3.8"` top-level key.

### Basic docker-compose.yml (server profile)

```yaml
services:
  manusclaw:
    build: .
    container_name: manusclaw-server
    restart: unless-stopped
    profiles:
      - server
    ports:
      - "8765:8765"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - MANUSCLAW_API_KEY=${MANUSCLAW_API_KEY}
    volumes:
      - manusclaw-config:/root/.manusclaw
      - manusclaw-workspace:/app/workspace
    command: manusclaw-server --host 0.0.0.0 --port 8765

volumes:
  manusclaw-config:
    driver: local
  manusclaw-workspace:
    driver: local
```

### Multi-profile docker-compose.yml with Ollama

This configuration uses Docker Compose **profiles** to let you start only the services you need:

```yaml
services:
  manusclaw:
    build: .
    container_name: manusclaw-server
    restart: unless-stopped
    profiles:
      - server
      - multi-agent
    ports:
      - "8765:8765"
      - "2222:2222"   # SSH gateway (multi-agent only)
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - OLLAMA_BASE_URL=http://ollama:11434
      - MANUSCLAW_API_KEY=${MANUSCLAW_API_KEY}
      - MANUSCLAW_SSH_ENABLED=true
      - MANUSCLAW_SSH_PORT=2222
      - MANUSCLAW_LOG_LEVEL=INFO
    volumes:
      - manusclaw-config:/root/.manusclaw
      - manusclaw-workspace:/app/workspace
      - ./config.toml:/root/.manusclaw/config.toml:ro
      - ./ssh_host_keys:/root/.manusclaw/ssh/:ro
    command: manusclaw-server --host 0.0.0.0 --port 8765
    depends_on:
      ollama:
        condition: service_started
    networks:
      - manusclaw-net

  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: unless-stopped
    profiles:
      - multi-agent
    # Uncomment for GPU support (requires NVIDIA Container Toolkit)
    # deploy:
    #   resources:
    #     reservations:
    #       devices:
    #         - driver: nvidia
    #           count: all
    #           capabilities: [gpu]
    volumes:
      - ollama-data:/root/.ollama
    networks:
      - manusclaw-net

volumes:
  manusclaw-config:
    driver: local
  manusclaw-workspace:
    driver: local
  ollama-data:
    driver: local

networks:
  manusclaw-net:
    driver: bridge
```

### Corresponding .env file

Create a `.env` file in the same directory as your `docker-compose.yml`:

```env
# API Keys
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Server authentication (v5)
MANUSCLAW_API_KEY=your-secure-api-key-change-this

# Logging
MANUSCLAW_LOG_LEVEL=INFO
```

### docker-compose commands with profiles

```bash
# Start only the server profile
docker compose --profile server up -d

# Start the full multi-agent stack (server + Ollama + SSH)
docker compose --profile multi-agent up -d

# Start and rebuild images
docker compose --profile server up -d --build

# View logs for all services
docker compose logs -f

# View logs for just ManusClaw
docker compose logs -f manusclaw

# Stop all services (across all profiles)
docker compose down

# Stop and remove volumes (⚠️ deletes all data)
docker compose down -v

# Restart a specific service
docker compose restart manusclaw

# Scale (run multiple ManusClaw instances)
docker compose up -d --scale manusclaw=3
```

### Pulling Ollama models

After starting the Ollama container, pull the models you need:

```bash
docker exec ollama ollama pull llama3
docker exec ollama ollama pull codellama
docker exec ollama ollama pull mistral
```

---

## VPS Deployment

Deploying ManusClaw directly on a VPS gives you maximum control over the environment and avoids Docker's overhead. This section covers a complete, production-ready VPS deployment.

### Step 1: Provision and secure your VPS

```bash
# Update the system
sudo apt update && sudo apt upgrade -y

# Create a dedicated user (don't run as root)
sudo useradd -m -s /bin/bash manusclaw
sudo usermod -aG sudo manusclaw

# Set up SSH key authentication
sudo mkdir -p /home/manusclaw/.ssh
sudo cp ~/.ssh/authorized_keys /home/manusclaw/.ssh/
sudo chown -R manusclaw:manusclaw /home/manusclaw/.ssh
sudo chmod 700 /home/manusclaw/.ssh
sudo chmod 600 /home/manusclaw/.ssh/authorized_keys

# Disable password authentication
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd

# Set up the firewall
sudo ufw allow OpenSSH
sudo ufw allow 8765/tcp   # ManusClaw v5 server port
sudo ufw allow 2222/tcp    # SSH gateway (optional, for multi-agent)
sudo ufw allow 80/tcp      # HTTP (for Nginx)
sudo ufw allow 443/tcp     # HTTPS (for Nginx)
sudo ufw --force enable
```

### Step 2: Install Python and ManusClaw

```bash
# Switch to the manusclaw user
sudo su - manusclaw

# Install Python 3.11+
sudo apt install -y software-properties-common
sudo add-apt-repository -y ppa:deadsnakes/ppa
sudo apt update
sudo apt install -y python3.11 python3.11-venv python3.11-dev python3-pip

# Create a virtual environment
python3.11 -m venv ~/manusclaw-env
source ~/manusclaw-env/bin/activate

# Install ManusClaw
pip install manusclaw

# Install Playwright browsers (optional)
playwright install --with-deps chromium

# Verify
manusclaw --version
```

### Step 3: Configure ManusClaw

```bash
# Create the config directory
mkdir -p ~/.manusclaw

# Create the .env file with API keys
cat > ~/.manusclaw/.env << 'EOF'
OPENAI_API_KEY=sk-proj-xxx
MANUSCLAW_API_KEY=your-secure-key
EOF

chmod 600 ~/.manusclaw/.env

# Create config.toml
cat > ~/.manusclaw/config.toml << 'EOF'
[llm]
provider = "openai"
model = "gpt-4o"

[server]
host = "127.0.0.1"
port = 8765
api_key = "your-secure-key"

[logging]
level = "INFO"
file = "/home/manusclaw/logs/manusclaw.log"
rotation = "10 MB"
retention = "7 days"

[permissions]
mode = "BUILD"

[workspace]
path = "/home/manusclaw/workspace"
auto_create = true
git_init = true
EOF

# Create directories
mkdir -p ~/workspace ~/logs
```

**Note:** Setting `host = "127.0.0.1"` ensures ManusClaw only accepts local connections. Remote access is handled by the Nginx reverse proxy, which adds SSL and additional security.

---

## Systemd Service Setup

Systemd ensures ManusClaw starts automatically, restarts on failure, and integrates with the OS's logging system.

### Create the service file

```bash
sudo nano /etc/systemd/system/manusclaw.service
```

Add the following content:

```ini
[Unit]
Description=ManusClaw AI Agent Server v5
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=manusclaw
Group=manusclaw
WorkingDirectory=/home/manusclaw

# Activate the virtual environment and run the server
ExecStart=/home/manusclaw/manusclaw-env/bin/manusclaw-server --host 127.0.0.1 --port 8765

# Restart on failure
Restart=on-failure
RestartSec=10
StartLimitBurst=5
StartLimitIntervalSec=300

# Environment
Environment=PATH=/home/manusclaw/manusclaw-env/bin:/usr/bin:/bin
EnvironmentFile=/home/manusclaw/.manusclaw/.env

# Security hardening
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=read-only
ReadWritePaths=/home/manusclaw/workspace /home/manusclaw/logs /home/manusclaw/.manusclaw
PrivateTmp=true

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=manusclaw

[Install]
WantedBy=multi-user.target
```

### Enable and start the service

```bash
# Reload systemd to pick up the new service
sudo systemctl daemon-reload

# Enable auto-start on boot
sudo systemctl enable manusclaw

# Start the service
sudo systemctl start manusclaw

# Check the status
sudo systemctl status manusclaw

# View logs
sudo journalctl -u manusclaw -f

# View recent logs (last 100 lines)
sudo journalctl -u manusclaw -n 100

# Restart the service
sudo systemctl restart manusclaw

# Stop the service
sudo systemctl stop manusclaw
```

### SSH gateway service (v5, optional)

If you're deploying the multi-agent profile with the SSH gateway, create a separate systemd service:

```bash
sudo nano /etc/systemd/system/manusclaw-ssh.service
```

```ini
[Unit]
Description=ManusClaw SSH Remote Gateway
After=network.target manusclaw.service
Wants=network-online.target

[Service]
Type=simple
User=manusclaw
Group=manusclaw
WorkingDirectory=/home/manusclaw

ExecStart=/home/manusclaw/manusclaw-env/bin/manusclaw-ssh start

Restart=on-failure
RestartSec=10

Environment=PATH=/home/manusclaw/manusclaw-env/bin:/usr/bin:/bin
EnvironmentFile=/home/manusclaw/.manusclaw/.env

StandardOutput=journal
StandardError=journal
SyslogIdentifier=manusclaw-ssh

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable manusclaw-ssh
sudo systemctl start manusclaw-ssh
```

### Cron service (optional)

If you use the cron scheduling feature, create a separate service:

```bash
sudo nano /etc/systemd/system/manusclaw-cron.service
```

```ini
[Unit]
Description=ManusClaw Cron Scheduler
After=network.target manusclaw.service
Wants=network-online.target

[Service]
Type=simple
User=manusclaw
Group=manusclaw
WorkingDirectory=/home/manusclaw

ExecStart=/home/manusclaw/manusclaw-env/bin/manusclaw-cron

Restart=on-failure
RestartSec=10

Environment=PATH=/home/manusclaw/manusclaw-env/bin:/usr/bin:/bin
EnvironmentFile=/home/manusclaw/.manusclaw/.env

StandardOutput=journal
StandardError=journal
SyslogIdentifier=manusclaw-cron

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable manusclaw-cron
sudo systemctl start manusclaw-cron
```

---

## Nginx Reverse Proxy

Nginx sits between the internet and your ManusClaw server, handling SSL termination, rate limiting, and request routing. This is the recommended setup for any publicly accessible deployment.

> **v5 Note:** The default port changed from 8000 to **8765** in v5.0.0. Update your Nginx configuration accordingly.

### Install Nginx

```bash
sudo apt install -y nginx
```

### Create the Nginx configuration

```bash
sudo nano /etc/nginx/sites-available/manusclaw
```

```nginx
# Upstream ManusClaw server (v5 default port)
upstream manusclaw_backend {
    server 127.0.0.1:8765;
    keepalive 64;
}

# HTTP → HTTPS redirect
server {
    listen 80;
    listen [::]:80;
    server_name manusclaw.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

# HTTPS server
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name manusclaw.yourdomain.com;

    # SSL certificates (configure after setting up certbot)
    ssl_certificate /etc/letsencrypt/live/manusclaw.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/manusclaw.yourdomain.com/privkey.pem;

    # SSL configuration (Mozilla Intermediate)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;

    # HSTS (optional but recommended)
    add_header Strict-Transport-Security "max-age=63072000" always;

    # Security headers
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options DENY;
    add_header X-XSS-Protection "1; mode=block";

    # Logging
    access_log /var/log/nginx/manusclaw_access.log;
    error_log /var/log/nginx/manusclaw_error.log;

    # Maximum request body size
    client_max_body_size 50M;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=manusclaw:10m rate=30r/m;
    limit_req zone=manusclaw burst=10 nodelay;

    # Proxy settings
    location / {
        proxy_pass http://manusclaw_backend;
        proxy_http_version 1.1;

        # WebSocket support (for canvas, webchat, v5 WebSocket endpoints)
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 300s;
        proxy_read_timeout 300s;

        # Buffering
        proxy_buffering off;
        proxy_request_buffering off;
    }

    # Health check endpoint (no rate limiting)
    location /health {
        proxy_pass http://manusclaw_backend/health;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
    }

    # Webhook endpoint — allow larger payloads and pass through signature headers
    location /webhooks/ {
        proxy_pass http://manusclaw_backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Signature $http_x_signature;
        proxy_set_header X-Webhook-Signature $http_x_webhook_signature;
    }
}
```

### Enable the configuration

```bash
# Create a symlink to enable the site
sudo ln -s /etc/nginx/sites-available/manusclaw /etc/nginx/sites-enabled/

# Remove the default site (optional)
sudo rm /etc/nginx/sites-enabled/default

# Test the configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

---

## SSL/TLS Setup

Use Let's Encrypt (via Certbot) to obtain free, automatically-renewing SSL certificates.

### Install Certbot

```bash
sudo apt install -y certbot python3-certbot-nginx
```

### Obtain a certificate

```bash
# Make sure your domain's DNS points to your server's IP
# Then run:
sudo certbot --nginx -d manusclaw.yourdomain.com
```

Certbot will:
1. Verify domain ownership via an HTTP challenge
2. Obtain an SSL certificate from Let's Encrypt
3. Modify the Nginx configuration to use the certificate
4. Set up automatic renewal

### Verify auto-renewal

```bash
# Check the renewal timer
sudo systemctl status certbot.timer

# Do a dry run to verify renewal works
sudo certbot renew --dry-run
```

Certificates are automatically renewed before they expire (every 60 days, with 30-day renewal window). No manual intervention is needed.

### Manual certificate setup (alternative)

If you're using a different certificate provider (not Let's Encrypt):

```bash
# Copy your certificate and key
sudo mkdir -p /etc/nginx/ssl
sudo cp your-cert.pem /etc/nginx/ssl/manusclaw.crt
sudo cp your-key.pem /etc/nginx/ssl/manusclaw.key
sudo chmod 600 /etc/nginx/ssl/manusclaw.key

# Update the Nginx config to point to these files
# ssl_certificate /etc/nginx/ssl/manusclaw.crt;
# ssl_certificate_key /etc/nginx/ssl/manusclaw.key;
```

---

## Auto-Start on Boot

ManusClaw should start automatically when the server boots. If you're using systemd (recommended), this is already handled by the `WantedBy=multi-user.target` line in the service file. Just make sure the service is enabled:

```bash
sudo systemctl enable manusclaw
```

If you're NOT using systemd, you can use alternative methods:

### Using crontab @reboot

```bash
# Edit the manusclaw user's crontab
crontab -e

# Add this line:
@reboot /home/manusclaw/manusclaw-env/bin/manusclaw-server --host 127.0.0.1 --port 8765 >> /home/manusclaw/logs/manusclaw.log 2>&1
```

### Using /etc/rc.local

```bash
sudo nano /etc/rc.local
```

```bash
#!/bin/bash
# Start ManusClaw server
su - manusclaw -c "/home/manusclaw/manusclaw-env/bin/manusclaw-server --host 127.0.0.1 --port 8765 >> /home/manusclaw/logs/manusclaw.log 2>&1 &"
exit 0
```

```bash
sudo chmod +x /etc/rc.local
```

---

## Background Execution

For situations where you need to run ManusClaw without systemd (e.g., on a shared server or for temporary tasks), these methods allow you to keep it running after you disconnect.

### nohup

The simplest approach — runs a command that persists after you log out:

```bash
nohup manusclaw-server --host 0.0.0.0 --port 8765 > manusclaw.log 2>&1 &

# Note the PID for later
echo $!

# To stop it later:
kill <PID>
```

**Pros:** Simple, available everywhere.  
**Cons:** No automatic restart on failure, limited monitoring.

### tmux

tmux provides a persistent terminal session that you can detach from and reattach to:

```bash
# Start a new tmux session
tmux new -s manusclaw

# Inside tmux, start ManusClaw
manusclaw-server --host 0.0.0.0 --port 8765

# Detach: Press Ctrl+B, then D
# The server continues running in the background

# Reattach later
tmux attach -t manusclaw

# List tmux sessions
tmux ls

# Kill the session
tmux kill-session -t manusclaw
```

**Pros:** Can see real-time output, easy to reattach, supports multiple panes.  
**Cons:** Requires tmux to be installed, session dies if the server reboots.

### screen

Similar to tmux but older and more widely available on minimal server installations:

```bash
# Start a screen session
screen -S manusclaw

# Inside screen, start ManusClaw
manusclaw-server --host 0.0.0.0 --port 8765

# Detach: Press Ctrl+A, then D
# Reattach later
screen -r manusclaw

# List screen sessions
screen -ls

# Kill the session
screen -X -S manusclaw quit
```

---

## Process Management with Supervisord

Supervisord is a process control system that provides automatic restarts, logging, and management for long-running processes. It's a good alternative to systemd for environments where you don't have root access or need more flexibility.

### Install Supervisord

```bash
sudo apt install -y supervisor
```

### Create the ManusClaw configuration

```bash
sudo nano /etc/supervisor/conf.d/manusclaw.conf
```

```ini
[program:manusclaw]
command=/home/manusclaw/manusclaw-env/bin/manusclaw-server --host 127.0.0.1 --port 8765
directory=/home/manusclaw
user=manusclaw
autostart=true
autorestart=true
startsecs=10
startretries=5
redirect_stderr=true
stdout_logfile=/home/manusclaw/logs/manusclaw-supervisor.log
stdout_logfile_maxbytes=10MB
stdout_logfile_backups=5
environment=PATH="/home/manusclaw/manusclaw-env/bin:/usr/bin:/bin",HOME="/home/manusclaw"
```

### Start and manage

```bash
# Reload supervisor configuration
sudo supervisorctl reread
sudo supervisorctl update

# Check status
sudo supervisorctl status manusclaw

# Restart
sudo supervisorctl restart manusclaw

# Stop
sudo supervisorctl stop manusclaw

# View logs
sudo tail -f /home/manusclaw/logs/manusclaw-supervisor.log
```

---

## Channel Adapter Deployment

ManusClaw v5.0.0 supports 12+ messaging channels. In production, each channel adapter runs alongside the ManusClaw server. Here's how to deploy them.

### Starting channel adapters

Channel adapters are started individually and connect to the running ManusClaw server:

```bash
# Start Telegram channel
manusclaw-channels start telegram

# Start Discord channel
manusclaw-channels start discord

# Start multiple channels
manusclaw-channels start telegram,discord,slack

# Start Matrix channel
manusclaw-channels start matrix
```

### Running channels as systemd services

For production deployments, run each channel as a separate systemd service:

```bash
# Create a generic channel service template
sudo nano /etc/systemd/system/manusclaw-channel@.service
```

```ini
[Unit]
Description=ManusClaw Channel — %i
After=network.target manusclaw.service
Wants=network-online.target

[Service]
Type=simple
User=manusclaw
Group=manusclaw
WorkingDirectory=/home/manusclaw

ExecStart=/home/manusclaw/manusclaw-env/bin/manusclaw-channels start %i

Restart=on-failure
RestartSec=10

Environment=PATH=/home/manusclaw/manusclaw-env/bin:/usr/bin:/bin
EnvironmentFile=/home/manusclaw/.manusclaw/.env

StandardOutput=journal
StandardError=journal
SyslogIdentifier=manusclaw-channel-%i

[Install]
WantedBy=multi-user.target
```

```bash
# Enable specific channels
sudo systemctl daemon-reload
sudo systemctl enable manusclaw-channel@telegram
sudo systemctl enable manusclaw-channel@discord
sudo systemctl start manusclaw-channel@telegram
sudo systemctl start manusclaw-channel@discord

# Check status
sudo systemctl status manusclaw-channel@telegram
```

### Channel adapter notes

| Channel | Environment Variables Required | Notes |
|---------|-------------------------------|-------|
| Telegram | `TELEGRAM_BOT_TOKEN` | Needs webhook setup for v5 server |
| Discord | `DISCORD_BOT_TOKEN` | Gateway (WebSocket) — no inbound ports needed |
| Slack | `SLACK_BOT_TOKEN` | Web API — polling or Socket Mode |
| WhatsApp | `WHATSAPP_ACCESS_TOKEN`, `WHATSAPP_BUSINESS_PHONE_ID` | Requires Meta Business verification |
| Matrix | `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN`, `MATRIX_USER_ID` | Long-polling — firewall-friendly |
| IRC | `IRC_SERVER`, `IRC_PORT`, `IRC_NICK`, `IRC_CHANNELS` | Pure TCP outbound |
| Twitch | `TWITCH_BOT_TOKEN`, `TWITCH_CHANNEL` | IRC-over-TLS outbound |
| Signal | `SIGNAL_CLI_REST_URL`, `SIGNAL_CLI_NUMBER` | Requires signal-cli daemon |
| WebChat | None | Built-in — available when server runs |
| SSH | `MANUSCLAW_SSH_ENABLED`, `MANUSCLAW_SSH_PORT` | Requires port 2222 open |

---

## Security Recommendations

Security is critical for any deployment, especially when ManusClaw has the ability to execute code and modify files. Follow these recommendations to secure your deployment.

### 1. Never expose ManusClaw directly to the internet

Always use a reverse proxy (Nginx) with SSL in front of ManusClaw. The server should bind to `127.0.0.1`, not `0.0.0.0`:

```toml
[server]
host = "127.0.0.1"  # Only accept local connections
```

### 2. Always set a server API key

In v5, the environment variable is `MANUSCLAW_API_KEY` (previously `MANUSCLAW_SERVER_API_KEY`):

```toml
[server]
api_key = "a-strong-random-key-at-least-32-characters"
```

Generate a strong key:

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(32))"
```

### 3. Use HTTPS exclusively

Never transmit API keys or conversation data over plain HTTP. Use Let's Encrypt certificates and redirect all HTTP traffic to HTTPS.

### 4. Restrict file system access

Run ManusClaw as a dedicated user with minimal permissions:

```bash
# Create a user that can't log in interactively
sudo useradd -r -s /usr/sbin/nologin manusclaw

# Set up the workspace with appropriate permissions
sudo mkdir -p /opt/manusclaw/workspace
sudo chown -R manusclaw:manusclaw /opt/manusclaw
sudo chmod 750 /opt/manusclaw
```

### 5. Use firewall rules

Only open the ports you need:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw --force enable
```

### 6. Keep API keys out of version control

```bash
# Add to .gitignore
echo ".env" >> .gitignore
echo "config.toml" >> .gitignore  # If it contains API keys
```

### 7. Use restrictive permission modes

In production, use PLAN mode or carefully configure which tools require confirmation:

```toml
[permissions]
mode = "PLAN"
deny = ["shell_exec"]  # Prevent shell execution in production
```

### 8. Regular updates

Keep ManusClaw and its dependencies up to date:

```bash
pip install --upgrade manusclaw
```

### 9. Audit logs

Enable file logging and review logs regularly:

```toml
[logging]
level = "INFO"
file = "/var/log/manusclaw/manusclaw.log"
rotation = "10 MB"
retention = "30 days"
```

### 10. Restrict CORS origins

Never use `["*"]` in production:

```toml
[server]
cors_origins = ["https://your-app.example.com"]
```

### 11. Webhook HMAC verification (v5)

All incoming webhooks should verify HMAC-SHA256 signatures:

```bash
# Create a webhook with a signing secret
manusclaw-webhook create \
  --url "/webhooks/github-push" \
  --secret "$(python3 -c 'import secrets; print(secrets.token_urlsafe(32))')" \
  --prompt "Analyze this GitHub push: {{payload.head_commit.message}}"
```

Never skip webhook signature verification in production. ManusClaw v5 rejects unsigned webhook requests by default.

### 12. SSH gateway security (v5)

The built-in SSH gateway (`manusclaw-ssh`) enforces:

- **Public key auth only** — no password authentication
- **Command whitelist** — only 9 approved commands (status, restart, logs, agent, channels, cron, help, exit)
- **Input validation** — rejects pipes, redirects, and shell metacharacters

Additional hardening:

```bash
# Use dedicated SSH host keys (not your system keys)
mkdir -p ~/.manusclaw/ssh
ssh-keygen -t ed25519 -f ~/.manusclaw/ssh/ssh_host_ed25519_key -N ""

# Restrict authorized SSH users
echo "ssh-ed25519 AAAA... your-key-here admin" > ~/.manusclaw/ssh/authorized_keys
```

### 13. Credential pool security

If using the credential pool for rate limit rotation, protect your keys:

```bash
# Set keys in the .env file (never in config.toml or a public repo)
cat >> ~/.manusclaw/.env << 'EOF'
OPENAI_API_KEY_1=sk-key1
OPENAI_API_KEY_2=sk-key2
OPENAI_API_KEY_3=sk-key3
EOF
chmod 600 ~/.manusclaw/.env
```

---

## Resource Requirements

Understanding resource requirements helps you choose the right infrastructure for your deployment.

### Minimum requirements (API-based providers)

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| **CPU** | 1 core | 2 cores |
| **RAM** | 512 MB | 1 GB |
| **Disk** | 1 GB | 5 GB |
| **Network** | 1 Mbps | 10 Mbps+ |

API-based providers (OpenAI, Anthropic, etc.) have minimal resource requirements because the heavy computation happens on the provider's servers.

### Requirements for local models (Ollama)

| Model Size | Minimum RAM | Recommended RAM | GPU VRAM |
|------------|------------|----------------|----------|
| 7B parameters | 8 GB | 16 GB | 6 GB (optional) |
| 13B parameters | 16 GB | 32 GB | 12 GB (recommended) |
| 34B parameters | 32 GB | 64 GB | 24 GB (required) |
| 70B parameters | 64 GB | 128 GB | 48 GB (required) |

### Requirements for GGUF models

GGUF models use quantization to reduce memory usage. The requirements depend on the quantization level:

| Quantization | Quality | Size (7B model) | RAM Needed |
|-------------|---------|-----------------|------------|
| Q4_K_M | Good | ~4 GB | ~6 GB |
| Q5_K_M | Better | ~5 GB | ~7 GB |
| Q6_K | Very good | ~6 GB | ~8 GB |
| Q8_0 | Excellent | ~7 GB | ~9 GB |
| F16 | Lossless | ~14 GB | ~16 GB |

### Disk space for Playwright browsers

If you install Playwright browsers (for web browsing capabilities):

| Browser | Disk Space |
|---------|-----------|
| Chromium | ~400 MB |
| Firefox | ~300 MB |
| WebKit | ~200 MB |

---

## Scaling Considerations

As your usage grows, you may need to scale your ManusClaw deployment. Here are strategies for different scaling challenges.

### Vertical scaling (bigger server)

The simplest approach — upgrade your VPS to a more powerful instance. This works well for single-user or small-team deployments.

- **CPU:** Upgrade to more cores for concurrent request handling
- **RAM:** Add more RAM for larger context windows or local models
- **Disk:** Use SSD storage for faster file operations
- **Network:** Upgrade bandwidth for high-throughput API usage

### Horizontal scaling (multiple instances)

Run multiple ManusClaw server instances behind a load balancer:

```nginx
upstream manusclaw_backend {
    server 127.0.0.1:8765;
    server 127.0.0.1:8766;
    server 127.0.0.1:8767;
}
```

Each instance runs independently with its own configuration. You'll need to ensure:

1. **Shared workspace** — Use NFS or a shared block storage for the workspace directory
2. **Session persistence** — Configure sticky sessions in your load balancer so a user's requests always go to the same instance
3. **Shared memory** — Use a shared MEMORY.md file or disable per-instance memory

### Using a credential pool for rate limits

If you're hitting API rate limits, add more API keys to the credential pool (see the [Configuration Guide](configuration.md#credential-pool)):

```env
OPENAI_API_KEY_1=sk-key1
OPENAI_API_KEY_2=sk-key2
OPENAI_API_KEY_3=sk-key3
```

### Model failover (v5)

v5's model failover profiles can help with scaling by automatically routing to alternative providers when one is overloaded:

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

### Caching strategies

For frequently asked questions or repeated operations, consider adding a caching layer:

1. **Response caching** — Cache LLM responses for identical queries (save tokens and cost)
2. **File caching** — Cache file reads to reduce disk I/O
3. **Search result caching** — Cache web search results to reduce API calls

### Database backend (advanced)

For large-scale deployments, consider replacing the file-based storage with a database:

- **Redis** for session storage and caching
- **PostgreSQL** for persistent task and memory storage
- **S3-compatible storage** for workspace files

These configurations require custom integration and are not provided out of the box by ManusClaw v5.0.0, but the architecture supports extension through the skills system.

### Monitoring

Set up monitoring for your production deployment:

```bash
# Install monitoring tools
sudo apt install -y prometheus-node-exporter

# Configure health check endpoint
# ManusClaw v5 provides /health endpoint when running in server mode
curl http://localhost:8765/health
# Returns: {"status": "ok", "version": "5.0.0"}
```

Use the health check endpoint with external monitoring services (UptimeRobot, Pingdom, etc.) to get alerts when your deployment is down.
