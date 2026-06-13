# Uninstall Guide — ManusClaw v5.1.0

This guide covers completely removing ManusClaw v5.1.0 from your system. Whether you're switching to a different tool, troubleshooting a persistent issue, or simply cleaning up, follow these steps to remove every trace of ManusClaw.

> **⚠️ Warning:** Some steps in this guide delete data permanently. Back up anything you want to keep before proceeding.

---

## Table of Contents

- [Quick Uninstall](#quick-uninstall)
- [Step 1: Uninstall the Python Package](#step-1-uninstall-the-python-package)
- [Step 2: Remove Configuration Files](#step-2-remove-configuration-files)
- [Step 3: Remove Workspace Data](#step-3-remove-workspace-data)
- [Step 4: Remove Docker Images and Containers](#step-4-remove-docker-images-and-containers)
- [Step 5: Remove Kubernetes Resources (v5.1)](#step-5-remove-kubernetes-resources-v51)
- [Step 6: Remove Playwright Browsers](#step-6-remove-playwright-browsers)
- [Step 7: Remove Virtual Environment](#step-7-remove-virtual-environment)
- [Step 8: Remove Environment Variables](#step-8-remove-environment-variables)
- [Step 9: Remove Systemd Services](#step-9-remove-systemd-services)
- [Step 10: Remove Cloud Resources (v5.1)](#step-10-remove-cloud-resources-v51)
- [Step 11: Clean All Data (Complete Purge)](#step-11-clean-all-data-complete-purge)
- [Back Up Before Uninstalling](#back-up-before-uninstalling)

---

## Quick Uninstall

If you just want to remove the ManusClaw program and don't care about cleaning up config files:

```bash
pip uninstall manusclaw -y
```

This removes the Python package but leaves behind configuration files, workspace data, and browser installations. For a complete removal, follow all steps below.

---

## Step 1: Uninstall the Python Package

```bash
# Standard uninstall
pip uninstall manusclaw -y

# If you installed in a virtual environment, activate it first
source ~/manusclaw-env/bin/activate
pip uninstall manusclaw -y
```

Verify the package is gone:

```bash
pip show manusclaw
# Should output: WARNING: Package(s) not found: manusclaw

which manusclaw
# Should output: manusclaw not found
```

If you installed from source:

```bash
cd /path/to/manusclaw
pip uninstall manusclaw -y
rm -rf /path/to/manusclaw
```

---

## Step 2: Remove Configuration Files

ManusClaw stores its configuration in `~/.manusclaw/`. This directory contains your `config.yaml`, `config.toml`, `.env` file (with API keys), memory files, sessions, and all v5.1 state data.

### Back up first (optional but recommended)

```bash
cp -r ~/.manusclaw ~/manusclaw-config-backup
```

### Remove the config directory

```bash
rm -rf ~/.manusclaw
```

### Selective removal

```bash
rm -f ~/.manusclaw/config.yaml         # v5.0+ primary config
rm -f ~/.manusclaw/config.toml          # Legacy config
rm -f ~/.manusclaw/.env                 # API keys and secrets
rm -f ~/.manusclaw/runtime.yaml         # v5.1 runtime overrides
rm -f ~/.manusclaw/MEMORY.md            # Long-term memory
rm -f ~/.manusclaw/USER.md              # User profile
rm -rf ~/.manusclaw/sessions            # Session data
rm -rf ~/.manusclaw/tasks               # Task data
rm -rf ~/.manusclaw/skills              # Custom skill definitions
rm -rf ~/.manusclaw/logs                # Log files
rm -rf ~/.manusclaw/profiles            # Config profiles
rm -rf ~/.manusclaw/ssh                 # SSH gateway keys
rm -f ~/.manusclaw/cron.yaml            # Cron job persistence
rm -rf ~/.manusclaw/nodes               # Canvas node state

# v5.1 directories
rm -rf ~/.manusclaw/hooks               # Hook scripts
rm -rf ~/.manusclaw/secrets             # Encrypted secrets / Vault cache
rm -rf ~/.manusclaw/migrations          # Migration state
rm -rf ~/.manusclaw/context             # Context management state
rm -rf ~/.manusclaw/conversations       # Conversation persistence
rm -rf ~/.manusclaw/plugins             # Integration plugins
rm -rf ~/.manusclaw/files               # File store (local)
```

### Also check for config files in other locations

```bash
rm -f ./config.yaml
rm -f ./config.toml
rm -f ./.env
sudo rm -f /etc/manusclaw/config.yaml
sudo rm -f /etc/manusclaw/config.toml
```

---

## Step 3: Remove Workspace Data

The workspace directory contains your project files that ManusClaw created or modified. **Be very careful here** — you may want to keep your project files.

### Check what's in the workspace

```bash
ls -la workspace/
```

### Back up important files

```bash
cp -r workspace/ ~/manusclaw-workspace-backup/
```

### Remove the workspace

```bash
# ⚠️ This deletes all files in the workspace directory
rm -rf workspace/
```

---

## Step 4: Remove Docker Images and Containers

If you ran ManusClaw in Docker, remove the containers, images, and volumes.

### Stop and remove containers

```bash
docker stop manusclaw-server 2>/dev/null
docker rm manusclaw-server 2>/dev/null

# If using docker-compose
cd /path/to/manusclaw
docker compose down

# With volumes
docker compose down -v
```

### Remove Docker images

```bash
docker images | grep manusclaw
docker rmi manusclaw:latest
docker rmi manusclaw:5.1.0
docker rmi manusclaw/manusclaw:5.1.0
docker image prune -f
```

### Remove Docker volumes

```bash
docker volume ls | grep manusclaw
docker volume rm manusclaw-config
docker volume rm manusclaw-workspace
docker volume prune -f
```

### Remove v5.1 enterprise containers

```bash
# Vault
docker stop manusclaw-vault 2>/dev/null
docker rm manusclaw-vault 2>/dev/null
docker volume rm vault-data 2>/dev/null

# OpenTelemetry Collector
docker stop manusclaw-otel 2>/dev/null
docker rm manusclaw-otel 2>/dev/null

# Prometheus
docker stop manusclaw-prometheus 2>/dev/null
docker rm manusclaw-prometheus 2>/dev/null
docker volume rm prometheus-data 2>/dev/null

# Grafana
docker stop manusclaw-grafana 2>/dev/null
docker rm manusclaw-grafana 2>/dev/null
docker volume rm grafana-data 2>/dev/null
```

### Remove the source repository

```bash
rm -rf /path/to/manusclaw
```

---

## Step 5: Remove Kubernetes Resources (v5.1)

If you deployed ManusClaw on Kubernetes:

```bash
# Uninstall Helm release
helm uninstall manusclaw --namespace manusclaw

# Delete the namespace
kubectl delete namespace manusclaw

# Delete secrets
kubectl delete secret manusclaw-secrets -n manusclaw 2>/dev/null

# Delete PVCs (persistent volume claims)
kubectl delete pvc -n manusclaw --all

# Remove Helm repository
helm repo remove manusclaw
```

### Remove manual Kubernetes resources

```bash
kubectl delete -f deployment.yaml
kubectl delete -f service.yaml
kubectl delete -f ingress.yaml
kubectl delete -f hpa.yaml
kubectl delete -f pdb.yaml
kubectl delete -f secrets.yaml
kubectl delete namespace manusclaw
```

---

## Step 6: Remove Playwright Browsers

```bash
playwright uninstall --all

# Remove the Playwright cache directory
rm -rf ~/Library/Caches/ms-playwright       # macOS
rm -rf ~/.cache/ms-playwright               # Linux
rm -rf "%LOCALAPPDATA%\ms-playwright"       # Windows
```

If you also want to remove the Playwright Python package:

```bash
pip uninstall playwright -y
```

---

## Step 7: Remove Virtual Environment

```bash
# Deactivate first (if active)
deactivate

# Remove the virtual environment directory
rm -rf ~/manusclaw-env

# Remove any aliases
# Edit ~/.bashrc or ~/.zshrc and remove lines like:
# source ~/manusclaw-env/bin/activate
# alias manusclaw="~/manusclaw-env/bin/manusclaw"
```

---

## Step 8: Remove Environment Variables

Clean up all ManusClaw-related environment variables from your shell profile.

### Remove from shell profile

```bash
nano ~/.bashrc    # Or ~/.zshrc on macOS

# Remove or comment out lines like:
# export OPENAI_API_KEY="sk-..."
# export ANTHROPIC_API_KEY="sk-ant-..."
# export MANUSCLAW_CONFIG_DIR="..."
# export MANUSCLAW_WORKSPACE="..."
# source ~/manusclaw-env/bin/activate

source ~/.bashrc
```

### Unset in the current session

```bash
# Core variables
unset OPENAI_API_KEY
unset ANTHROPIC_API_KEY
unset GOOGLE_API_KEY
unset MISTRAL_API_KEY
unset GROQ_API_KEY
unset HUGGINGFACE_API_KEY
unset OPENROUTER_API_KEY
unset TOGETHER_API_KEY
unset DEEPINFRA_API_KEY
unset COHERE_API_KEY
unset AWS_ACCESS_KEY_ID
unset AWS_SECRET_ACCESS_KEY
unset AZURE_OPENAI_API_KEY

# ManusClaw-specific
unset MANUSCLAW_CONFIG_DIR
unset MANUSCLAW_WORKSPACE
unset MANUSCLAW_LOG_LEVEL
unset MANUSCLAW_API_KEY
unset MANUSCLAW_SERVER_API_KEY
unset MANUSCLAW_SSH_ENABLED
unset MANUSCLAW_SSH_PORT
unset MANUSCLAW_PROVIDER
unset MANUSCLAW_MODEL
unset MANUSCLAW_PROFILE

# Voice
unset PICOVOICE_API_KEY
unset ELEVENLABS_API_KEY

# Gmail
unset GMAIL_WATCH_TOPIC_NAME
unset GMAIL_AUTO_REPLY

# v5.1 Enterprise
unset VAULT_ADDR
unset VAULT_TOKEN
unset VAULT_ROLE_ID
unset VAULT_SECRET_ID
unset OTEL_EXPORTER_OTLP_ENDPOINT
unset OTEL_SERVICE_NAME
unset AWS_DEFAULT_REGION
unset GOOGLE_APPLICATION_CREDENTIALS
unset AZURE_STORAGE_CONNECTION_STRING
unset GITHUB_TOKEN
unset GITLAB_TOKEN
unset BITBUCKET_USERNAME
unset BITBUCKET_APP_PASSWORD
unset JIRA_API_TOKEN
unset NOTION_API_KEY
unset PAGERDUTY_API_KEY

# Channel tokens
unset TELEGRAM_BOT_TOKEN
unset DISCORD_BOT_TOKEN
unset SLACK_BOT_TOKEN
unset WHATSAPP_ACCESS_TOKEN
unset WHATSAPP_BUSINESS_PHONE_ID
unset SIGNAL_CLI_REST_URL
unset SIGNAL_CLI_NUMBER
unset MATRIX_HOMESERVER
unset MATRIX_ACCESS_TOKEN
unset MATRIX_USER_ID
unset TWITCH_BOT_TOKEN
unset TWITCH_CHANNEL
unset MICROSOFT_APP_ID
unset MICROSOFT_APP_PASSWORD
unset GOOGLE_CHAT_SERVICE_ACCOUNT
unset LINE_CHANNEL_SECRET
unset LINE_CHANNEL_ACCESS_TOKEN

# Credential pool keys
unset OPENAI_API_KEY_1 OPENAI_API_KEY_2 OPENAI_API_KEY_3
unset ANTHROPIC_API_KEY_1 ANTHROPIC_API_KEY_2
```

---

## Step 9: Remove Systemd Services

If you set up ManusClaw as a systemd service:

```bash
# Stop the services
sudo systemctl stop manusclaw
sudo systemctl stop manusclaw-cron
sudo systemctl stop manusclaw-ssh

# Disable auto-start
sudo systemctl disable manusclaw
sudo systemctl disable manusclaw-cron
sudo systemctl disable manusclaw-ssh

# Remove the service files
sudo rm /etc/systemd/system/manusclaw.service
sudo rm /etc/systemd/system/manusclaw-cron.service
sudo rm /etc/systemd/system/manusclaw-ssh.service
sudo rm /etc/systemd/system/manusclaw-channel@.service

# Reload systemd
sudo systemctl daemon-reload
```

Also remove Nginx configuration if set up:

```bash
sudo rm /etc/nginx/sites-available/manusclaw
sudo rm /etc/nginx/sites-enabled/manusclaw
sudo nginx -t && sudo systemctl reload nginx
```

---

## Step 10: Remove Cloud Resources (v5.1)

If you deployed ManusClaw to a cloud provider, remove the cloud resources:

### AWS ECS / Fargate

```bash
# Delete ECS service
aws ecs delete-service --cluster manusclaw --service manusclaw --force

# Delete ECS cluster
aws ecs delete-cluster --cluster manusclaw

# Delete task definition (deregister all revisions)
aws ecs deregister-task-definition --task-definition manusclaw:1

# Delete CloudWatch log group
aws logs delete-log-group --log-group-name /ecs/manusclaw

# Delete Secrets Manager secrets
aws secretsmanager delete-secret --secret-id manusclaw/openai-api-key --force-delete-without-recovery
aws secretsmanager delete-secret --secret-id manusclaw/anthropic-api-key --force-delete-without-recovery
```

### Google Cloud Run

```bash
# Delete Cloud Run service
gcloud run services delete manusclaw --region us-central1 --quiet

# Delete container image
gcloud container images delete gcr.io/PROJECT_ID/manusclaw:5.1.0 --quiet

# Delete Secret Manager secrets
gcloud secrets delete openai-api-key --quiet
gcloud secrets delete anthropic-api-key --quiet
```

### Azure Container Instances

```bash
# Delete container instance
az container delete --resource-group manusclaw-rg --name manusclaw --yes

# Delete resource group (removes all resources in it)
az group delete --name manusclaw-rg --yes
```

### S3 Bucket (File Store)

```bash
# Empty and delete the S3 bucket
aws s3 rm s3://manusclaw-artifacts --recursive
aws s3 rb s3://manusclaw-artifacts --force
```

---

## Step 11: Clean All Data (Complete Purge)

This is the nuclear option. It removes absolutely everything related to ManusClaw. **Make sure you've backed up anything important before running these commands.**

```bash
#!/bin/bash
# Complete ManusClaw v5.1.0 purge script
# ⚠️ THIS REMOVES ALL MANUSCLAW DATA PERMANENTLY ⚠️

echo "This will completely remove ManusClaw v5.1.0 and all its data."
echo "Press Ctrl+C to cancel, or Enter to continue."
read

# 1. Uninstall the package
pip uninstall manusclaw -y 2>/dev/null

# 2. Remove configuration
rm -rf ~/.manusclaw

# 3. Remove workspace
rm -rf ./workspace

# 4. Remove Playwright
playwright uninstall --all 2>/dev/null
rm -rf ~/.cache/ms-playwright
rm -rf ~/Library/Caches/ms-playwright

# 5. Remove virtual environment
rm -rf ~/manusclaw-env

# 6. Remove pip cache
pip cache purge

# 7. Remove Docker resources
docker stop manusclaw-server 2>/dev/null
docker rm manusclaw-server 2>/dev/null
docker rmi manusclaw:latest manusclaw:5.1.0 manusclaw/manusclaw:5.1.0 2>/dev/null
docker volume rm manusclaw-config manusclaw-workspace 2>/dev/null

# 8. Remove v5.1 Docker containers
docker stop manusclaw-vault manusclaw-otel manusclaw-prometheus manusclaw-grafana 2>/dev/null
docker rm manusclaw-vault manusclaw-otel manusclaw-prometheus manusclaw-grafana 2>/dev/null
docker volume rm vault-data prometheus-data grafana-data 2>/dev/null

# 9. Remove source code (if cloned)
rm -rf ~/manusclaw

# 10. Remove Python cache
find ~ -type d -name "__pycache__" -path "*manusclaw*" -exec rm -rf {} + 2>/dev/null

# 11. Remove systemd services
sudo systemctl stop manusclaw manusclaw-cron manusclaw-ssh 2>/dev/null
sudo systemctl disable manusclaw manusclaw-cron manusclaw-ssh 2>/dev/null
sudo rm /etc/systemd/system/manusclaw.service 2>/dev/null
sudo rm /etc/systemd/system/manusclaw-cron.service 2>/dev/null
sudo rm /etc/systemd/system/manusclaw-ssh.service 2>/dev/null
sudo rm /etc/systemd/system/manusclaw-channel@.service 2>/dev/null
sudo systemctl daemon-reload 2>/dev/null

echo "ManusClaw v5.1.0 has been completely removed."
```

Save this as `purge_manusclaw.sh` and run:

```bash
chmod +x purge_manusclaw.sh
./purge_manusclaw.sh
```

---

## Back Up Before Uninstalling

If you want to preserve your ManusClaw data before removing it:

```bash
#!/bin/bash
BACKUP_DIR=~/manusclaw-backup-$(date +%Y%m%d_%H%M%S)
mkdir -p "$BACKUP_DIR"

# Back up configuration
cp -r ~/.manusclaw "$BACKUP_DIR/config" 2>/dev/null

# Back up workspace
cp -r ./workspace "$BACKUP_DIR/workspace" 2>/dev/null

# Back up environment variables
env | grep -E '(OPENAI|ANTHROPIC|GOOGLE|MISTRAL|AWS_|HUGGINGFACE|OPENROUTER|MANUSCLAW|OLLAMA|VAULT|OTEL|GITHUB|GITLAB|BITBUCKET|JIRA|NOTION|PAGERDUTY|TELEGRAM|DISCORD|SLACK|WHATSAPP|SIGNAL|MATRIX|TWITCH|LINE|PICOVOICE|ELEVENLABS)' > "$BACKUP_DIR/env_vars.txt" 2>/dev/null

# Back up Docker volumes
docker run --rm -v manusclaw-config:/data -v "$BACKUP_DIR":/backup alpine tar czf /backup/config-volume.tar.gz -C /data . 2>/dev/null
docker run --rm -v manusclaw-workspace:/data -v "$BACKUP_DIR":/backup alpine tar czf /backup/workspace-volume.tar.gz -C /data . 2>/dev/null

echo "Backup complete: $BACKUP_DIR"
ls -la "$BACKUP_DIR"
```
