# Deployment Guide — ManusClaw v5.1.0

This guide covers deploying ManusClaw v5.1.0 in production environments. Whether you're running it on a VPS, in Docker, on Kubernetes, or in the cloud, this guide provides battle-tested configurations for reliable, secure, and maintainable deployments.

---

## Table of Contents

- [Deployment Overview](#deployment-overview)
- [Docker Deployment](#docker-deployment)
  - [Single Container](#single-container)
  - [Docker Compose (Production)](#docker-compose-production)
  - [docker-compose Reference](#docker-compose-reference)
- [Kubernetes Deployment](#kubernetes-deployment)
  - [Helm Chart](#helm-chart)
  - [Manual Kubernetes Manifests](#manual-kubernetes-manifests)
  - [Auto-Scaling](#auto-scaling)
  - [Pod Disruption Budgets](#pod-disruption-budgets)
- [Cloud Deployment](#cloud-deployment)
  - [AWS ECS / Fargate](#aws-ecs--fargate)
  - [Google Cloud Run](#google-cloud-run)
  - [Azure Container Instances](#azure-container-instances)
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
| **Docker Compose** | Multi-service deployments with profiles | Medium | Medium |
| **Kubernetes** | Large-scale, auto-scaling, enterprise | High | High |
| **AWS ECS/Fargate** | AWS-native serverless containers | Medium | High |
| **Google Cloud Run** | GCP-native serverless containers | Low | High |
| **Azure Container Instances** | Azure-native simple containers | Low | Medium |
| **VPS (bare metal)** | Maximum control, custom tuning | Medium | Low-Medium |

For most users and small teams, Docker Compose is the right choice. Kubernetes is only necessary if you need auto-scaling, rolling updates, or enterprise-grade orchestration. Cloud-managed container services are ideal if you want zero infrastructure management.

### v5.1 Deployment Modes

ManusClaw v5.1.0 supports three deployment modes:

| Mode | Description | Best For |
|------|-------------|----------|
| **Server** | HTTP/WebSocket server with API | Production, multi-user |
| **REPL** | Interactive command-line interface | Development, personal use |
| **Single-shot** | One prompt, one response | CI/CD, scripts, cron |

---

## Docker Deployment

### Single Container

The simplest Docker deployment — a single container running the ManusClaw server:

```bash
docker run -d \
  --name manusclaw \
  --restart unless-stopped \
  -p 8765:8765 \
  -p 2222:2222 \
  -e OPENAI_API_KEY=sk-proj-your-key \
  -e ANTHROPIC_API_KEY=sk-ant-your-key \
  -v ~/.manusclaw:/root/.manusclaw \
  manusclaw/manusclaw:5.1.0
```

**With v5.1 enterprise features:**

```bash
docker run -d \
  --name manusclaw \
  --restart unless-stopped \
  -p 8765:8765 \
  -p 2222:2222 \
  -p 9090:9090 \
  -e OPENAI_API_KEY=sk-proj-your-key \
  -e VAULT_ADDR=http://vault:8200 \
  -e VAULT_TOKEN=your-vault-token \
  -e OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317 \
  -e AWS_ACCESS_KEY_ID=your-aws-key \
  -e AWS_SECRET_ACCESS_KEY=your-aws-secret \
  -e AWS_DEFAULT_REGION=us-east-1 \
  -e GITHUB_TOKEN=your-github-token \
  -v ~/.manusclaw:/root/.manusclaw \
  manusclaw/manusclaw:5.1.0
```

### Docker Compose (Production)

A production-ready Docker Compose configuration with health checks, auto-restart, and enterprise services:

```yaml
# docker-compose.yml — ManusClaw v5.1.0 Production
version: "3.8"

services:
  # ── Core ManusClaw Server ───────────────────────────
  manusclaw:
    image: manusclaw/manusclaw:5.1.0
    container_name: manusclaw-server
    restart: unless-stopped
    ports:
      - "8765:8765"
    volumes:
      - manusclaw-config:/root/.manusclaw
    env_file: .env
    environment:
      - MANUSCLAW_SERVER_HOST=0.0.0.0
      - MANUSCLAW_SERVER_PORT=8765
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8765/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s
    deploy:
      resources:
        limits:
          memory: 2G
          cpus: "2.0"
        reservations:
          memory: 512M
          cpus: "0.5"

  # ── SSH Gateway ─────────────────────────────────────
  manusclaw-ssh:
    image: manusclaw/manusclaw:5.1.0
    container_name: manusclaw-ssh
    restart: unless-stopped
    command: manusclaw-ssh start
    ports:
      - "2222:2222"
    volumes:
      - manusclaw-config:/root/.manusclaw
    env_file: .env
    environment:
      - MANUSCLAW_SSH_ENABLED=true
      - MANUSCLAW_SSH_PORT=2222
    depends_on:
      manusclaw:
        condition: service_healthy

  # ── Channel Adapters ────────────────────────────────
  manusclaw-telegram:
    image: manusclaw/manusclaw:5.1.0
    container_name: manusclaw-telegram
    restart: unless-stopped
    command: manusclaw-channels start telegram
    volumes:
      - manusclaw-config:/root/.manusclaw
    env_file: .env
    depends_on:
      manusclaw:
        condition: service_healthy

  manusclaw-discord:
    image: manusclaw/manusclaw:5.1.0
    container_name: manusclaw-discord
    restart: unless-stopped
    command: manusclaw-channels start discord
    volumes:
      - manusclaw-config:/root/.manusclaw
    env_file: .env
    depends_on:
      manusclaw:
        condition: service_healthy

  # ── Cron Scheduler ──────────────────────────────────
  manusclaw-cron:
    image: manusclaw/manusclaw:5.1.0
    container_name: manusclaw-cron
    restart: unless-stopped
    command: manusclaw-cron start
    volumes:
      - manusclaw-config:/root/.manusclaw
    env_file: .env
    depends_on:
      manusclaw:
        condition: service_healthy

  # ── HashiCorp Vault (Enterprise) ────────────────────
  vault:
    image: hashicorp/vault:1.15
    container_name: manusclaw-vault
    restart: unless-stopped
    ports:
      - "8200:8200"
    environment:
      VAULT_DEV_ROOT_TOKEN_ID: "dev-only-token"
      VAULT_DEV_LISTEN_ADDRESS: "0.0.0.0:8200"
    cap_add:
      - IPC_LOCK
    volumes:
      - vault-data:/vault/data
    profiles: ["enterprise"]

  # ── OpenTelemetry Collector (Observability) ─────────
  otel-collector:
    image: otel/opentelemetry-collector-contrib:0.96.0
    container_name: manusclaw-otel
    restart: unless-stopped
    ports:
      - "4317:4317"   # OTLP gRPC
      - "4318:4318"   # OTLP HTTP
    volumes:
      - ./otel-config.yaml:/etc/otelcol-contrib/config.yaml
    profiles: ["observability"]

  # ── Prometheus (Observability) ──────────────────────
  prometheus:
    image: prom/prometheus:v2.50.0
    container_name: manusclaw-prometheus
    restart: unless-stopped
    ports:
      - "9090:9090"
    volumes:
      - prometheus-data:/prometheus
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    profiles: ["observability"]

  # ── Grafana (Observability) ─────────────────────────
  grafana:
    image: grafana/grafana:10.3.0
    container_name: manusclaw-grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: "admin"
    volumes:
      - grafana-data:/var/lib/grafana
    profiles: ["observability"]

volumes:
  manusclaw-config:
  vault-data:
  prometheus-data:
  grafana-data:
```

### docker-compose Reference

#### Starting Services

```bash
# Start core services only
docker compose up -d

# Start with SSH gateway
docker compose up -d manusclaw manusclaw-ssh

# Start with channels
docker compose up -d manusclaw manusclaw-telegram manusclaw-discord

# Start with enterprise services (Vault)
docker compose --profile enterprise up -d

# Start with observability stack
docker compose --profile observability up -d

# Start everything
docker compose --profile enterprise --profile observability up -d
```

#### Managing Services

```bash
# View logs
docker compose logs -f manusclaw

# View logs for a specific service
docker compose logs -f manusclaw-telegram

# Restart a service
docker compose restart manusclaw

# Scale channel adapters (multiple Telegram bots)
docker compose up -d --scale manusclaw-telegram=3

# Stop all services
docker compose down

# Stop and remove volumes (full reset)
docker compose down -v
```

---

## Kubernetes Deployment

### Helm Chart

ManusClaw v5.1.0 includes a Helm chart for Kubernetes deployment:

```bash
# Add the ManusClaw Helm repository
helm repo add manusclaw https://manusagents.github.io/manusclaw-helm
helm repo update

# Install with default values
helm install manusclaw manusclaw/manusclaw \
  --namespace manusclaw \
  --create-namespace

# Install with custom values
helm install manusclaw manusclaw/manusclaw \
  --namespace manusclaw \
  --create-namespace \
  -f values.yaml
```

**values.yaml:**

```yaml
# ManusClaw Helm Chart values.yaml

replicaCount: 2

image:
  repository: manusclaw/manusclaw
  tag: "5.1.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 8765

ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  hosts:
    - host: manusclaw.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: manusclaw-tls
      hosts:
        - manusclaw.example.com

resources:
  limits:
    cpu: "2"
    memory: 2Gi
  requests:
    cpu: "500m"
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

# v5.1 Features
ssh:
  enabled: true
  port: 2222
  service:
    type: LoadBalancer
    port: 2222

channels:
  telegram:
    enabled: true
  discord:
    enabled: true

enterprise:
  vault:
    enabled: true
    address: "http://vault:8200"

observability:
  enabled: true
  otlp:
    endpoint: "http://otel-collector:4317"
  prometheus:
    enabled: true
    serviceMonitor:
      enabled: true

secrets:
  # Reference existing Kubernetes secrets
  openaiApiKey:
    name: manusclaw-secrets
    key: openai-api-key
  anthropicApiKey:
    name: manusclaw-secrets
    key: anthropic-api-key

podDisruptionBudget:
  enabled: true
  minAvailable: 1

nodeSelector: {}
tolerations: []
affinity: {}
```

### Manual Kubernetes Manifests

If you prefer not to use Helm, you can deploy with raw Kubernetes manifests.

**Namespace and secrets:**

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: manusclaw
---
# secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: manusclaw-secrets
  namespace: manusclaw
type: Opaque
stringData:
  openai-api-key: "sk-proj-your-key"
  anthropic-api-key: "sk-ant-your-key"
  server-api-key: "your-secure-server-key"
```

**Deployment:**

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: manusclaw
  namespace: manusclaw
  labels:
    app: manusclaw
spec:
  replicas: 2
  selector:
    matchLabels:
      app: manusclaw
  template:
    metadata:
      labels:
        app: manusclaw
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
    spec:
      containers:
        - name: manusclaw
          image: manusclaw/manusclaw:5.1.0
          ports:
            - containerPort: 8765
              name: http
            - containerPort: 9090
              name: metrics
          env:
            - name: OPENAI_API_KEY
              valueFrom:
                secretKeyRef:
                  name: manusclaw-secrets
                  key: openai-api-key
            - name: ANTHROPIC_API_KEY
              valueFrom:
                secretKeyRef:
                  name: manusclaw-secrets
                  key: anthropic-api-key
            - name: MANUSCLAW_SERVER_HOST
              value: "0.0.0.0"
          resources:
            limits:
              cpu: "2"
              memory: 2Gi
            requests:
              cpu: "500m"
              memory: 512Mi
          livenessProbe:
            httpGet:
              path: /health
              port: 8765
            initialDelaySeconds: 30
            periodSeconds: 30
          readinessProbe:
            httpGet:
              path: /health
              port: 8765
            initialDelaySeconds: 10
            periodSeconds: 10
          volumeMounts:
            - name: config
              mountPath: /root/.manusclaw
      volumes:
        - name: config
          persistentVolumeClaim:
            claimName: manusclaw-config
```

**Service and Ingress:**

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: manusclaw
  namespace: manusclaw
spec:
  selector:
    app: manusclaw
  ports:
    - name: http
      port: 8765
      targetPort: 8765
    - name: metrics
      port: 9090
      targetPort: 9090
---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: manusclaw
  namespace: manusclaw
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/websocket-services: manusclaw
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - manusclaw.example.com
      secretName: manusclaw-tls
  rules:
    - host: manusclaw.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: manusclaw
                port:
                  number: 8765
```

### Auto-Scaling

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: manusclaw
  namespace: manusclaw
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: manusclaw
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

### Pod Disruption Budgets

```yaml
# pdb.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: manusclaw
  namespace: manusclaw
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: manusclaw
```

---

## Cloud Deployment

### AWS ECS / Fargate

Deploy ManusClaw on AWS ECS with Fargate for serverless container execution.

**Task Definition:**

```json
{
  "family": "manusclaw",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "1024",
  "memory": "2048",
  "containerDefinitions": [
    {
      "name": "manusclaw",
      "image": "manusclaw/manusclaw:5.1.0",
      "essential": true,
      "portMappings": [
        { "containerPort": 8765, "protocol": "tcp" }
      ],
      "environment": [
        { "name": "MANUSCLAW_SERVER_HOST", "value": "0.0.0.0" }
      ],
      "secrets": [
        { "name": "OPENAI_API_KEY", "valueFrom": "arn:aws:secretsmanager:region:account:secret:manusclaw/openai-api-key" },
        { "name": "ANTHROPIC_API_KEY", "valueFrom": "arn:aws:secretsmanager:region:account:secret:manusclaw/anthropic-api-key" }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/manusclaw",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:8765/health || exit 1"],
        "interval": 30,
        "timeout": 10,
        "retries": 3,
        "startPeriod": 30
      }
    }
  ]
}
```

**Deploy with CLI:**

```bash
# Create ECS cluster
aws ecs create-cluster --cluster-name manusclaw

# Register task definition
aws ecs register-task-definition --cli-input-json file://task-definition.json

# Create service
aws ecs create-service \
  --cluster manusclaw \
  --service-name manusclaw \
  --task-definition manusclaw \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxx],securityGroups=[sg-xxx],assignPublicIp=ENABLED}"
```

### Google Cloud Run

Deploy ManusClaw on Google Cloud Run for a fully managed, auto-scaling container platform.

```bash
# Build and push the container
gcloud builds submit --tag gcr.io/PROJECT_ID/manusclaw:5.1.0

# Deploy to Cloud Run
gcloud run deploy manusclaw \
  --image gcr.io/PROJECT_ID/manusclaw:5.1.0 \
  --platform managed \
  --region us-central1 \
  --memory 2Gi \
  --cpu 2 \
  --min-instances 1 \
  --max-instances 10 \
  --set-env-vars "MANUSCLAW_SERVER_HOST=0.0.0.0" \
  --set-secrets "OPENAI_API_KEY=openai-api-key:latest,ANTHROPIC_API_KEY=anthropic-api-key:latest" \
  --allow-unauthenticated

# For authenticated access:
gcloud run deploy manusclaw \
  --no-allow-unauthenticated \
  --set-env-vars "MANUSCLAW_SERVER_API_KEY=your-secure-key"
```

### Azure Container Instances

Deploy ManusClaw on Azure Container Instances for simple, fast container deployment.

```bash
# Create a resource group
az group create --name manusclaw-rg --location eastus

# Create container instance
az container create \
  --resource-group manusclaw-rg \
  --name manusclaw \
  --image manusclaw/manusclaw:5.1.0 \
  --ports 8765 \
  --cpu 2 \
  --memory 2 \
  --environment-variables MANUSCLAW_SERVER_HOST=0.0.0.0 \
  --secure-environment-variables \
    OPENAI_API_KEY=sk-proj-your-key \
    ANTHROPIC_API_KEY=sk-ant-your-key \
  --dns-name-label manusclaw-unique

# Get the FQDN
az container show \
  --resource-group manusclaw-rg \
  --name manusclaw \
  --query ipAddress.fqdn
```

---

## VPS Deployment

### Systemd Service Setup

Create a systemd service for running ManusClaw as a background daemon:

```ini
# /etc/systemd/system/manusclaw.service
[Unit]
Description=ManusClaw AI Agent Server
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=manusclaw
Group=manusclaw
WorkingDirectory=/opt/manusclaw
ExecStart=/opt/manusclaw/env/bin/manusclaw-server --host 0.0.0.0 --port 8765
Restart=on-failure
RestartSec=10
StandardOutput=journal
StandardError=journal

# Security hardening
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=read-only
ReadWritePaths=/opt/manusclaw /var/log/manusclaw
PrivateTmp=true

# Environment
EnvironmentFile=/opt/manusclaw/.env

[Install]
WantedBy=multi-user.target
```

**Setup commands:**

```bash
# Create the manusclaw user
sudo useradd -r -s /bin/false -d /opt/manusclaw manusclaw

# Install and configure
sudo mkdir -p /opt/manusclaw
sudo python3 -m venv /opt/manusclaw/env
sudo /opt/manusclaw/env/bin/pip install "manusclaw[all]"

# Copy the service file
sudo cp manusclaw.service /etc/systemd/system/

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable manusclaw
sudo systemctl start manusclaw

# Check status
sudo systemctl status manusclaw
```

**Additional service files:**

```ini
# /etc/systemd/system/manusclaw-ssh.service
[Unit]
Description=ManusClaw SSH Gateway
After=manusclaw.service
Requires=manusclaw.service

[Service]
Type=simple
User=manusclaw
Group=manusclaw
WorkingDirectory=/opt/manusclaw
ExecStart=/opt/manusclaw/env/bin/manusclaw-ssh start
Restart=on-failure
RestartSec=10
EnvironmentFile=/opt/manusclaw/.env

[Install]
WantedBy=multi-user.target
```

```ini
# /etc/systemd/system/manusclaw-channel@.service
# Template for channel adapters
[Unit]
Description=ManusClaw Channel Adapter (%i)
After=manusclaw.service
Requires=manusclaw.service

[Service]
Type=simple
User=manusclaw
Group=manusclaw
WorkingDirectory=/opt/manusclaw
ExecStart=/opt/manusclaw/env/bin/manusclaw-channels start %i
Restart=on-failure
RestartSec=10
EnvironmentFile=/opt/manusclaw/.env

[Install]
WantedBy=multi-user.target
```

```bash
# Start a channel adapter
sudo systemctl enable manusclaw-channel@telegram
sudo systemctl start manusclaw-channel@telegram
```

### Nginx Reverse Proxy

```nginx
# /etc/nginx/sites-available/manusclaw
upstream manusclaw {
    server 127.0.0.1:8765;
    keepalive 64;
}

server {
    listen 80;
    server_name manusclaw.example.com;

    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name manusclaw.example.com;

    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/manusclaw.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/manusclaw.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # API proxy
    location / {
        proxy_pass http://manusclaw;
        proxy_http_version 1.1;

        # WebSocket support
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

    # Health check endpoint (no auth required)
    location /health {
        proxy_pass http://manusclaw/health;
        access_log off;
    }

    # Prometheus metrics (restrict access)
    location /metrics {
        allow 10.0.0.0/8;
        allow 172.16.0.0/12;
        deny all;
        proxy_pass http://manusclaw/metrics;
    }
}
```

### SSL/TLS Setup

Using Let's Encrypt with Certbot:

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d manusclaw.example.com

# Auto-renewal (Certbot adds a cron job automatically)
sudo certbot renew --dry-run
```

### Auto-Start on Boot

With systemd (recommended):

```bash
sudo systemctl enable manusclaw
sudo systemctl enable manusclaw-ssh
sudo systemctl enable manusclaw-channel@telegram
```

---

## Background Execution

For simple deployments without systemd:

### Using nohup

```bash
nohup manusclaw-server --host 0.0.0.0 --port 8765 > manusclaw.log 2>&1 &
echo $! > manusclaw.pid
```

### Using screen

```bash
screen -dmS manusclaw manusclaw-server --host 0.0.0.0 --port 8765
screen -r manusclaw  # Reattach
```

### Using tmux

```bash
tmux new-session -d -s manusclaw 'manusclaw-server --host 0.0.0.0 --port 8765'
tmux attach -t manusclaw  # Reattach
```

---

## Process Management with Supervisord

```ini
# /etc/supervisor/conf.d/manusclaw.conf
[program:manusclaw]
command=/opt/manusclaw/env/bin/manusclaw-server --host 0.0.0.0 --port 8765
directory=/opt/manusclaw
user=manusclaw
autostart=true
autorestart=true
startsecs=10
startretries=3
stopwaitsecs=30
stdout_logfile=/var/log/manusclaw/server.log
stdout_logfile_maxbytes=50MB
stdout_logfile_backups=10
stderr_logfile=/var/log/manusclaw/error.log
environment=HOME="/opt/manusclaw",USER="manusclaw"

[program:manusclaw-ssh]
command=/opt/manusclaw/env/bin/manusclaw-ssh start
directory=/opt/manusclaw
user=manusclaw
autostart=true
autorestart=true
stdout_logfile=/var/log/manusclaw/ssh.log
stderr_logfile=/var/log/manusclaw/ssh-error.log
```

---

## Channel Adapter Deployment

Each messaging channel can run as a separate process, allowing independent scaling and isolation.

```bash
# Start channel adapters individually
manusclaw-channels start telegram &
manusclaw-channels start discord &
manusclaw-channels start slack &
manusclaw-channels start whatsapp &
```

### Multi-Channel Docker Compose

```yaml
# channels-only compose
version: "3.8"
services:
  telegram:
    image: manusclaw/manusclaw:5.1.0
    command: manusclaw-channels start telegram
    env_file: .env
    restart: unless-stopped

  discord:
    image: manusclaw/manusclaw:5.1.0
    command: manusclaw-channels start discord
    env_file: .env
    restart: unless-stopped

  slack:
    image: manusclaw/manusclaw:5.1.0
    command: manusclaw-channels start slack
    env_file: .env
    restart: unless-stopped
```

---

## Security Recommendations

### Production Security Checklist

- [ ] Enable RBAC with appropriate roles
- [ ] Enable input validation with strict mode
- [ ] Enable rate limiting
- [ ] Enable audit logging
- [ ] Use HashiCorp Vault or cloud secret managers for API keys
- [ ] Enable TLS/SSL for all endpoints
- [ ] Set `MANUSCLAW_SERVER_API_KEY` for API authentication
- [ ] Use SSH public key authentication only
- [ ] Restrict Prometheus metrics endpoint to internal networks
- [ ] Run containers as non-root user
- [ ] Set resource limits on containers/pods
- [ ] Enable network policies in Kubernetes
- [ ] Use encrypted storage for file store
- [ ] Enable secret rotation
- [ ] Regularly update ManusClaw and dependencies

### Network Security

```yaml
# Kubernetes NetworkPolicy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: manusclaw
  namespace: manusclaw
spec:
  podSelector:
    matchLabels:
      app: manusclaw
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: ingress-nginx
      ports:
        - port: 8765
  egress:
    - to: []  # Allow all egress (LLM APIs need this)
```

---

## Resource Requirements

### Minimum Resources

| Component | CPU | RAM | Disk |
|-----------|-----|-----|------|
| ManusClaw server | 0.5 core | 512 MB | 500 MB |
| SSH gateway | 0.1 core | 128 MB | 50 MB |
| Channel adapter (each) | 0.25 core | 256 MB | 50 MB |
| Cron scheduler | 0.1 core | 128 MB | 50 MB |

### Recommended Production Resources

| Component | CPU | RAM | Disk |
|-----------|-----|-----|------|
| ManusClaw server | 2 cores | 2 GB | 2 GB |
| SSH gateway | 0.5 core | 256 MB | 100 MB |
| Channel adapter (each) | 0.5 core | 512 MB | 100 MB |
| Cron scheduler | 0.25 core | 256 MB | 100 MB |
| Vault | 0.5 core | 256 MB | 1 GB |
| OTEL Collector | 0.5 core | 512 MB | 500 MB |
| Prometheus | 1 core | 1 GB | 10 GB |
| Grafana | 0.5 core | 256 MB | 1 GB |

---

## Scaling Considerations

### Horizontal Scaling

ManusClaw can scale horizontally by running multiple server instances behind a load balancer:

1. **Stateless server instances** — Keep session data in a shared store (database or Redis)
2. **Channel adapters** — Each channel adapter can be scaled independently
3. **Worker pools** — Use the parallel executor for concurrent task processing

### Vertical Scaling

For single-instance deployments, increase resources based on:

- **More users** → More RAM for session management
- **More channels** → More CPU for message processing
- **Larger contexts** → More RAM for context management
- **Parallel tasks** → More CPU cores for the parallel executor

### Session Persistence

When scaling horizontally, configure session persistence:

```yaml
conversation:
  persistence:
    backend: "database"  # Use database instead of file for multi-instance
```

### Cron Job Coordination

When running multiple instances, enable distributed cron scheduling to prevent duplicate execution:

```yaml
# In cron.yaml
distributed: true
lock_backend: "database"  # database | redis
```
