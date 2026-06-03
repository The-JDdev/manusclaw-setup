# Uninstall Guide — ManusClaw v4.0.0

This guide covers completely removing ManusClaw from your system. Whether you're switching to a different tool, troubleshooting a persistent issue, or simply cleaning up, follow these steps to remove every trace of ManusClaw.

> **⚠️ Warning:** Some steps in this guide delete data permanently. Back up anything you want to keep before proceeding.

---

## Table of Contents

- [Quick Uninstall](#quick-uninstall)
- [Step 1: Uninstall the Python Package](#step-1-uninstall-the-python-package)
- [Step 2: Remove Configuration Files](#step-2-remove-configuration-files)
- [Step 3: Remove Workspace Data](#step-3-remove-workspace-data)
- [Step 4: Remove Docker Images and Containers](#step-4-remove-docker-images-and-containers)
- [Step 5: Remove Playwright Browsers](#step-5-remove-playwright-browsers)
- [Step 6: Remove Virtual Environment](#step-6-remove-virtual-environment)
- [Step 7: Remove Environment Variables](#step-7-remove-environment-variables)
- [Step 8: Remove Systemd Services](#step-8-remove-systemd-services)
- [Step 9: Clean All Data (Complete Purge)](#step-9-clean-all-data-complete-purge)
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

Remove the ManusClaw package using pip:

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

If you installed from source using `pip install -e .`:

```bash
# Navigate to the source directory
cd /path/to/manusclaw

# Uninstall
pip uninstall manusclaw -y

# Remove the source code
rm -rf /path/to/manusclaw
```

---

## Step 2: Remove Configuration Files

ManusClaw stores its configuration in `~/.manusclaw/`. This directory contains your `config.toml`, `.env` file (with API keys), memory files, sessions, and more.

### Back up first (optional but recommended)

```bash
# Back up the entire config directory
cp -r ~/.manusclaw ~/manusclaw-config-backup

# Or back up specific files
mkdir -p ~/manusclaw-backup
cp ~/.manusclaw/config.toml ~/manusclaw-backup/
cp ~/.manusclaw/.env ~/manusclaw-backup/
cp ~/.manusclaw/MEMORY.md ~/manusclaw-backup/
cp ~/.manusclaw/USER.md ~/manusclaw-backup/
```

### Remove the config directory

```bash
# Remove everything
rm -rf ~/.manusclaw

# Or selectively remove:
rm -f ~/.manusclaw/config.toml
rm -f ~/.manusclaw/.env
rm -f ~/.manusclaw/MEMORY.md
rm -f ~/.manusclaw/USER.md
rm -rf ~/.manusclaw/sessions
rm -rf ~/.manusclaw/tasks
rm -rf ~/.manusclaw/skills
rm -rf ~/.manusclaw/logs
```

### Also check for config files in other locations

```bash
# Check current directory
rm -f ./config.toml
rm -f ./.env

# Check system-wide (if you installed there)
sudo rm -f /etc/manusclaw/config.toml
```

---

## Step 3: Remove Workspace Data

The workspace directory contains your project files that ManusClaw created or modified. **Be very careful here** — you may want to keep your project files even if you're removing ManusClaw.

### Check what's in the workspace

```bash
ls -la workspace/
```

### Back up important files

```bash
# Back up the entire workspace
cp -r workspace/ ~/manusclaw-workspace-backup/

# Or back up specific files
cp workspace/src/main.py ~/manusclaw-backup/
```

### Remove the workspace

```bash
# ⚠️ This deletes all files in the workspace directory
rm -rf workspace/

# If you configured a custom workspace path, remove that instead
# Check your config.toml for the workspace path if you're not sure
```

---

## Step 4: Remove Docker Images and Containers

If you ran ManusClaw in Docker, you need to remove the containers, images, and volumes separately.

### Stop and remove containers

```bash
# Stop running containers
docker stop manusclaw-server

# Remove containers
docker rm manusclaw-server

# If using docker-compose
cd /path/to/manusclaw
docker compose down
```

### Remove Docker images

```bash
# List ManusClaw images
docker images | grep manusclaw

# Remove specific images
docker rmi manusclaw:latest
docker rmi manusclaw:4.0.0

# Remove all dangling images (unused by any container)
docker image prune -f
```

### Remove Docker volumes

```bash
# List volumes
docker volume ls | grep manusclaw

# Remove specific volumes
docker volume rm manusclaw-config
docker volume rm manusclaw-workspace

# ⚠️ Remove all unused volumes
docker volume prune -f
```

### Remove the source repository (if cloned)

```bash
rm -rf /path/to/manusclaw
```

---

## Step 5: Remove Playwright Browsers

Playwright installs browser binaries that can take up significant disk space (~400 MB for Chromium).

```bash
# Uninstall all Playwright browsers
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

## Step 6: Remove Virtual Environment

If you created a dedicated virtual environment for ManusClaw, remove it:

```bash
# Deactivate first (if active)
deactivate

# Remove the virtual environment directory
rm -rf ~/manusclaw-env

# Remove any aliases you created
# Edit ~/.bashrc or ~/.zshrc and remove lines like:
# source ~/manusclaw-env/bin/activate
# alias manusclaw="~/manusclaw-env/bin/manusclaw"
```

---

## Step 7: Remove Environment Variables

Clean up any API keys or ManusClaw-related environment variables from your shell profile.

### Remove from shell profile

```bash
# Edit your shell profile
nano ~/.bashrc    # Or ~/.zshrc on macOS

# Remove or comment out lines like:
# export OPENAI_API_KEY="sk-..."
# export ANTHROPIC_API_KEY="sk-ant-..."
# export MANUSCLAW_CONFIG_DIR="..."
# export MANUSCLAW_WORKSPACE="..."
# source ~/manusclaw-env/bin/activate

# Save and reload
source ~/.bashrc
```

### Unset in the current session

```bash
unset OPENAI_API_KEY
unset ANTHROPIC_API_KEY
unset GOOGLE_API_KEY
unset MISTRAL_API_KEY
unset AWS_ACCESS_KEY_ID
unset AWS_SECRET_ACCESS_KEY
unset HUGGINGFACE_API_KEY
unset OPENROUTER_API_KEY
unset MANUSCLAW_CONFIG_DIR
unset MANUSCLAW_WORKSPACE
unset MANUSCLAW_LOG_LEVEL
unset MANUSCLAW_SERVER_API_KEY
unset MANUSCLAW_PROVIDER
unset MANUSCLAW_MODEL
```

---

## Step 8: Remove Systemd Services

If you set up ManusClaw as a systemd service (for VPS deployments), remove the service files:

```bash
# Stop the services
sudo systemctl stop manusclaw
sudo systemctl stop manusclaw-cron

# Disable auto-start
sudo systemctl disable manusclaw
sudo systemctl disable manusclaw-cron

# Remove the service files
sudo rm /etc/systemd/system/manusclaw.service
sudo rm /etc/systemd/system/manusclaw-cron.service

# Reload systemd
sudo systemctl daemon-reload

# Verify they're removed
systemctl status manusclaw
# Should show: Unit manusclaw.service could not be found
```

Also remove Nginx configuration if you set up a reverse proxy:

```bash
sudo rm /etc/nginx/sites-available/manusclaw
sudo rm /etc/nginx/sites-enabled/manusclaw
sudo nginx -t && sudo systemctl reload nginx
```

---

## Step 9: Clean All Data (Complete Purge)

This is the nuclear option. It removes absolutely everything related to ManusClaw. **Make sure you've backed up anything important before running these commands.**

```bash
#!/bin/bash
# Complete ManusClaw purge script
# ⚠️ THIS REMOVES ALL MANUSCLAW DATA PERMANENTLY ⚠️

echo "This will completely remove ManusClaw and all its data."
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
docker rmi manusclaw:latest manusclaw:4.0.0 2>/dev/null
docker volume rm manusclaw-config manusclaw-workspace 2>/dev/null

# 8. Remove source code (if cloned)
rm -rf ~/manusclaw

# 9. Remove Python cache
find ~ -type d -name "__pycache__" -path "*manusclaw*" -exec rm -rf {} + 2>/dev/null

echo "ManusClaw has been completely removed."
```

Save this as `purge_manusclaw.sh` and run:

```bash
chmod +x purge_manusclaw.sh
./purge_manusclaw.sh
```

---

## Back Up Before Uninstalling

If you want to preserve your ManusClaw data before removing it, here's a comprehensive backup script:

```bash
#!/bin/bash
# Back up all ManusClaw data
BACKUP_DIR=~/manusclaw-backup-$(date +%Y%m%d_%H%M%S)
mkdir -p "$BACKUP_DIR"

# Back up configuration
cp -r ~/.manusclaw "$BACKUP_DIR/config" 2>/dev/null

# Back up workspace (if it exists)
cp -r ./workspace "$BACKUP_DIR/workspace" 2>/dev/null

# Back up environment variables
env | grep -E '(OPENAI|ANTHROPIC|GOOGLE|MISTRAL|AWS_|HUGGINGFACE|OPENROUTER|MANUSCLAW|OLLAMA)' > "$BACKUP_DIR/env_vars.txt" 2>/dev/null

# Back up Docker volumes (if using Docker)
docker run --rm -v manusclaw-config:/data -v "$BACKUP_DIR":/backup alpine tar czf /backup/config-volume.tar.gz -C /data . 2>/dev/null
docker run --rm -v manusclaw-workspace:/data -v "$BACKUP_DIR":/backup alpine tar czf /backup/workspace-volume.tar.gz -C /data . 2>/dev/null

echo "Backup complete: $BACKUP_DIR"
echo "Contents:"
ls -la "$BACKUP_DIR"
```

This creates a timestamped backup directory containing all your configuration, workspace, and environment variables.
