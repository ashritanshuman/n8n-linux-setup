# n8n on Ubuntu — Fresh Docker Setup Guide

# Before these steps install docker from docker-linux-repo

> A full start-to-finish guide to run a local n8n server using Docker on Ubuntu.

**Compatible with:** i5 10th Gen+ · 8 GB RAM · Ubuntu 22.04 / 24.04

**What you get:**

| Feature | Status |
|---|---|
| Local n8n server | ✅ |
| Persistent storage | ✅ |
| Password protection | ✅ |
| Templates enabled | ✅ |
| Auto-start on boot | ✅ |
| Docker-based setup | ✅ |

---

## Table of Contents

1. [Update Ubuntu](#1-update-ubuntu)
2. [Install Required Packages](#2-install-required-packages)
3. [Add Docker Official GPG Key](#3-add-docker-official-gpg-key)
4. [Add Docker Repository](#4-add-docker-repository)
5. [Install Docker + Docker Compose](#5-install-docker--docker-compose)
6. [Enable Docker Without sudo](#6-enable-docker-without-sudo)
7. [Verify Docker Installation](#7-verify-docker-installation)
8. [Create n8n Project Folder](#8-create-n8n-project-folder)
9. [Create Persistent Storage Folder](#9-create-persistent-storage-folder)
10. [Create Environment File](#10-create-environment-file)
11. [Generate Encryption Key](#11-generate-encryption-key)
12. [Create Docker Compose File](#12-create-docker-compose-file)
13. [Start n8n](#13-start-n8n)
14. [Check Container Status](#14-check-container-status)
15. [View Logs](#15-view-logs)
16. [Open n8n in Browser](#16-open-n8n-in-browser)
17. [Hard Refresh Browser](#17-hard-refresh-browser)
18. [Auto-Start on Boot](#18-auto-start-on-boot)
19. [Useful Docker Commands](#19-useful-docker-commands)
20. [Update n8n](#20-update-n8n)
21. [Backup Your Workflows](#21-backup-your-workflows)
22. [Install Community Nodes](#22-install-community-nodes-optional)
23. [Recommended Next Steps](#23-recommended-next-steps)

---

## 1. Update Ubuntu

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2. Install Required Packages

```bash
sudo apt install -y ca-certificates curl gnupg lsb-release
```

---

## 3. Add Docker Official GPG Key

```bash
sudo mkdir -p /etc/apt/keyrings
```

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

---

## 4. Add Docker Repository

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

## 5. Install Docker + Docker Compose

```bash
sudo apt update
```

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

---

## 6. Enable Docker Without sudo

```bash
sudo usermod -aG docker $USER
```

Apply changes immediately:

```bash
newgrp docker
```

---

## 7. Verify Docker Installation

```bash
docker run hello-world
```

A success message confirms Docker is properly installed.

---

## 8. Create n8n Project Folder

```bash
mkdir ~/n8n
cd ~/n8n
```

---

## 9. Create Persistent Storage Folder

```bash
mkdir n8n_data
```

This folder stores your workflows, credentials, settings, templates, and installed nodes. Without it, all data is lost on container restart.

---

## 10. Create Environment File

```bash
nano .env
```

Paste the following, then customise the values marked below:

```env
# =========================
# BASIC SECURITY
# =========================

N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=yourpassword        # Change this

# =========================
# ENCRYPTION
# =========================

N8N_ENCRYPTION_KEY=replace_this_with_random_key   # See Step 11

# =========================
# TIMEZONE
# =========================

GENERIC_TIMEZONE=Asia/Kolkata
TZ=Asia/Kolkata

# =========================
# ENABLE TEMPLATES
# =========================

N8N_TEMPLATES_ENABLED=true
N8N_ONBOARDING_FLOW_DISABLED=false
N8N_PUBLIC_API_DISABLED=false
N8N_VERSION_NOTIFICATIONS_ENABLED=true
N8N_DIAGNOSTICS_ENABLED=true

# =========================
# PERFORMANCE FOR 8GB RAM
# =========================

EXECUTIONS_PROCESS=main
N8N_RUNNERS_ENABLED=false

# =========================
# OPTIONAL
# =========================

N8N_HIRING_BANNER_ENABLED=false
```

Save: `Ctrl + O` → Enter → `Ctrl + X`

---

## 11. Generate Encryption Key

Generate a random key:

```bash
openssl rand -hex 16
```

Example output:

```
3f8b9d6c2f1a7e4d9c5a8b1e6f2d3c4
```

Copy the output and replace the placeholder in `.env`:

```env
N8N_ENCRYPTION_KEY=your_generated_key_here
```

---

## 12. Create Docker Compose File

```bash
nano docker-compose.yml
```

Paste:

```yaml
version: "3.9"

services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    env_file:
      - .env
    environment:
      - NODE_ENV=production
    volumes:
      - ./n8n_data:/home/node/.n8n
```

Save and exit.

---

## 13. Start n8n

```bash
docker compose up -d
```

---

## 14. Check Container Status

```bash
docker ps
```

You should see `n8n` listed with status `Up`.

---

## 15. View Logs

```bash
docker compose logs -f
```

Wait until you see:

```
Editor is now accessible via:
http://localhost:5678
```

Press `Ctrl + C` to exit the log view.

---

## 16. Open n8n in Browser

Navigate to:

```
http://localhost:5678
```

Login with:

- **Username:** `admin`
- **Password:** `yourpassword` (or whatever you set in `.env`)

---

## 17. Hard Refresh Browser

Do a hard refresh to ensure templates load correctly:

```
Ctrl + Shift + R
```

Templates and community workflows should now be visible.

---

## 18. Auto-Start on Boot

No extra configuration needed. The `restart: unless-stopped` directive in `docker-compose.yml` ensures n8n starts automatically after every reboot.

---

## 19. Useful Docker Commands

| Action | Command |
|---|---|
| Stop n8n | `docker compose down` |
| Start n8n | `docker compose up -d` |
| Restart n8n | `docker compose restart` |
| View live logs | `docker compose logs -f` |

---

## 20. Update n8n

```bash
cd ~/n8n
docker compose pull
docker compose up -d
```

---

## 21. Backup Your Workflows

**Create a backup:**

```bash
tar -czf n8n_backup.tar.gz ~/n8n/n8n_data
```

**Restore from backup:**

```bash
tar -xzf n8n_backup.tar.gz
```

---

## 22. Install Community Nodes (Optional)

Inside n8n, go to **Settings → Community Nodes → Install**.

Popular community nodes:

- LangChain
- AI Agents
- Telegram
- WhatsApp
- Puppeteer
- Scraping tools

---

## 23. Recommended Next Steps

After your local setup is running, consider one of these:

1. Expose n8n to the internet via **Cloudflare Tunnel**
2. Connect a **Telegram bot**
3. Build **AI agents**
4. Automate **Gmail / Sheets / WhatsApp**
5. Run local AI with **Ollama**

---

## Notes for 8 GB RAM

**Works well:**

- Standard automations and API integrations
- Bots and scheduled tasks
- AI workflows using external APIs (OpenAI, Gemini, etc.)

**Avoid:**

- Heavy local LLMs running inside Docker
- Massive parallel scraping jobs
- 100+ concurrent workflow executions

---

## Final Folder Structure

```
~/n8n
 ├── docker-compose.yml
 ├── .env
 └── n8n_data/
```

This is a production-style local deployment of n8n running on Docker.
