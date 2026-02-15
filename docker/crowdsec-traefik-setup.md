# CrowdSec + Traefik Docker Setup Guide

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Step 1: Directory Structure](#step-1-directory-structure)
- [Step 2: Traefik Configuration](#step-2-traefik-configuration)
- [Step 3: CrowdSec Configuration](#step-3-crowdsec-configuration)
- [Step 4: Docker Compose](#step-4-docker-compose)
- [Step 5: Enroll with CrowdSec Console](#step-5-enroll-with-crowdsec-console)
- [Step 6: Register the Bouncer](#step-6-register-the-bouncer)
- [Step 7: Deploy the Stack](#step-7-deploy-the-stack)
- [Step 8: Verify the Setup](#step-8-verify-the-setup)
- [Step 9: Manage Collections and Scenarios](#step-9-manage-collections-and-scenarios)
- [Step 10: Maintenance and Operations](#step-10-maintenance-and-operations)
- [Troubleshooting](#troubleshooting)
- [Security Hardening](#security-hardening)

---

## Overview

**CrowdSec** is an open-source, collaborative intrusion prevention system (IPS) that analyzes logs, detects attacks, and shares threat intelligence across its community. When paired with **Traefik** (a modern reverse proxy/ingress controller), CrowdSec can protect your services by:

- Parsing Traefik access logs for malicious patterns
- Blocking bad IPs via a bouncer middleware in Traefik
- Sharing and receiving threat intelligence from the CrowdSec community

### How It Works

```
Internet → Traefik → CrowdSec Bouncer (middleware) → Your Services
                ↓
         Access Logs → CrowdSec Engine → Decisions (ban/captcha)
                                ↑
                    Community Blocklists
```

1. **Traefik** receives incoming requests and writes access logs.
2. **CrowdSec Engine (LAPI)** reads those logs, parses them, and detects attacks using scenarios.
3. When an attack is detected, CrowdSec creates a **decision** (e.g., ban the IP).
4. The **CrowdSec Bouncer** (Traefik plugin) queries CrowdSec's LAPI and blocks banned IPs before they reach your services.

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Docker Host                       │
│                                                     │
│  ┌──────────────┐     ┌──────────────────────────┐  │
│  │   CrowdSec   │     │        Traefik           │  │
│  │   Engine     │◄────│  (reads access logs)     │  │
│  │   (LAPI)     │     │                          │  │
│  │   :8080      │     │  :80 (HTTP)              │  │
│  └──────┬───────┘     │  :443 (HTTPS)            │  │
│         │             │  :8080 (Dashboard)        │  │
│         │ API         │                          │  │
│         ▼             │  ┌────────────────────┐  │  │
│  ┌──────────────┐     │  │ CrowdSec Bouncer   │  │  │
│  │  Decisions   │────►│  │ (Traefik Plugin)   │  │  │
│  │  Database    │     │  └────────────────────┘  │  │
│  └──────────────┘     └──────────────────────────┘  │
│                                                     │
│  ┌──────────────┐     ┌──────────────────────────┐  │
│  │  Service A   │     │  Service B               │  │
│  └──────────────┘     └──────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

**Components:**

| Component | Role | Port |
|-----------|------|------|
| **Traefik** | Reverse proxy / ingress controller | 80, 443, 8080 |
| **CrowdSec Engine** | Log parser, scenario engine, LAPI server | 8080 (internal), 6060 (metrics) |
| **CrowdSec Bouncer** | Traefik middleware plugin that queries LAPI and blocks IPs | N/A (embedded in Traefik) |

---

## Prerequisites

- Docker Engine 20.10+ and Docker Compose v2+
- A domain name (or local testing setup)
- Basic familiarity with Traefik and Docker networking
- (Optional) A CrowdSec Console account at [app.crowdsec.net](https://app.crowdsec.net)

---

## Step 1: Directory Structure

Create the project directory structure:

```bash
mkdir -p ~/crowdsec-traefik/{traefik,crowdsec,logs}
cd ~/crowdsec-traefik
```

Final structure:

```
crowdsec-traefik/
├── docker-compose.yml
├── traefik/
│   ├── traefik.yml              # Static configuration
│   ├── dynamic/
│   │   ├── middlewares.yml       # CrowdSec bouncer middleware
│   │   └── tls.yml              # (Optional) TLS settings
│   └── acme.json                # Let's Encrypt certificates
├── crowdsec/
│   ├── acquis.yaml              # Log acquisition config
│   └── profiles.yaml            # (Optional) Custom profiles
└── logs/
    └── traefik-access.log       # Traefik access logs (shared volume)
```

Create the directories and placeholder files:

```bash
mkdir -p traefik/dynamic crowdsec
touch traefik/acme.json
chmod 600 traefik/acme.json
touch logs/traefik-access.log
```

---

## Step 2: Traefik Configuration

### 2a. Static Configuration (`traefik/traefik.yml`)

```yaml
# traefik/traefik.yml

# API Dashboard
api:
  dashboard: true
  insecure: true  # Set to false in production; use auth middleware

# Entry Points
entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
  websecure:
    address: ":443"

# Certificate Resolvers (Let's Encrypt)
certificatesResolvers:
  letsencrypt:
    acme:
      email: your-email@example.com    # <-- Change this
      storage: /etc/traefik/acme.json
      httpChallenge:
        entryPoint: web

# Providers
providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
    network: proxy
  file:
    directory: /etc/traefik/dynamic
    watch: true

# Access Logging — CRITICAL for CrowdSec
accessLog:
  filePath: /var/log/traefik/traefik-access.log
  format: json
  fields:
    headers:
      defaultMode: keep
      names:
        User-Agent: keep
        X-Forwarded-For: keep

# Application Logging
log:
  level: INFO

# Experimental: CrowdSec Bouncer Plugin
experimental:
  plugins:
    crowdsec-bouncer-traefik-plugin:
      moduleName: github.com/maxlerebourg/crowdsec-bouncer-traefik-plugin
      version: v1.3.5    # Check for latest version
```

**Key points:**
- `accessLog` must be enabled with `json` format for CrowdSec to parse.
- The `experimental.plugins` section loads the CrowdSec bouncer as a Traefik plugin.
- HTTP-to-HTTPS redirection is configured on the `web` entrypoint.

### 2b. Dynamic Configuration — Middlewares (`traefik/dynamic/middlewares.yml`)

```yaml
# traefik/dynamic/middlewares.yml

http:
  middlewares:
    crowdsec:
      plugin:
        crowdsec-bouncer-traefik-plugin:
          enabled: true
          crowdsecLapiScheme: http
          crowdsecLapiHost: crowdsec:8080
          crowdsecLapiKey: "${CROWDSEC_BOUNCER_API_KEY}"  # Set after registration
          forwardedHeadersTrustedIPs:
            - "10.0.0.0/8"
            - "172.16.0.0/12"
            - "192.168.0.0/16"
          clientTrustedIPs:
            - "10.0.0.0/8"
            - "172.16.0.0/12"
            - "192.168.0.0/16"
          crowdsecMode: live          # live, stream, or alone
          updateIntervalSeconds: 15
          defaultDecisionSeconds: 300
```

> **Note:** The `crowdsecLapiKey` value will be generated in [Step 6](#step-6-register-the-bouncer). You can leave a placeholder for now and update it later.

### 2c. (Optional) TLS Configuration (`traefik/dynamic/tls.yml`)

```yaml
# traefik/dynamic/tls.yml

tls:
  options:
    default:
      minVersion: VersionTLS12
      cipherSuites:
        - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
        - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
        - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
      sniStrict: true
```

---

## Step 3: CrowdSec Configuration

### 3a. Acquisition Configuration (`crowdsec/acquis.yaml`)

This tells CrowdSec where to find logs and how to parse them:

```yaml
# crowdsec/acquis.yaml

filenames:
  - /var/log/traefik/*
labels:
  type: traefik
```

CrowdSec will tail all files in `/var/log/traefik/` and parse them using the `traefik` log parser (installed via the `crowdsecurity/traefik` collection).

### 3b. (Optional) Custom Profiles (`crowdsec/profiles.yaml`)

Override the default decision behavior:

```yaml
# crowdsec/profiles.yaml

name: default_ip_remediation
filters:
  - Alert.Remediation == true && Alert.GetScope() == "Ip"
decisions:
  - type: ban
    duration: 4h
on_success: break
---
name: default_range_remediation
filters:
  - Alert.Remediation == true && Alert.GetScope() == "Range"
decisions:
  - type: ban
    duration: 4h
on_success: break
```

---

## Step 4: Docker Compose

### `docker-compose.yml`

```yaml
# docker-compose.yml

networks:
  proxy:
    name: proxy
    driver: bridge
  crowdsec:
    name: crowdsec
    driver: bridge

volumes:
  crowdsec-db:
  crowdsec-config:
  traefik-logs:

services:
  # ─────────────────────────────────────────
  # Traefik - Reverse Proxy
  # ─────────────────────────────────────────
  traefik:
    image: traefik:v3.3
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"     # Dashboard (restrict in production)
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik/traefik.yml:/etc/traefik/traefik.yml:ro
      - ./traefik/dynamic:/etc/traefik/dynamic:ro
      - ./traefik/acme.json:/etc/traefik/acme.json
      - traefik-logs:/var/log/traefik
    networks:
      - proxy
      - crowdsec
    depends_on:
      - crowdsec
    labels:
      - "traefik.enable=true"
      # Dashboard (optional, secure in production)
      - "traefik.http.routers.traefik-dashboard.rule=Host(`traefik.example.com`)"
      - "traefik.http.routers.traefik-dashboard.entrypoints=websecure"
      - "traefik.http.routers.traefik-dashboard.tls.certresolver=letsencrypt"
      - "traefik.http.routers.traefik-dashboard.service=api@internal"
      - "traefik.http.routers.traefik-dashboard.middlewares=crowdsec@file"

  # ─────────────────────────────────────────
  # CrowdSec - Security Engine
  # ─────────────────────────────────────────
  crowdsec:
    image: crowdsecurity/crowdsec:latest
    container_name: crowdsec
    restart: unless-stopped
    environment:
      # Collections to install on first run
      COLLECTIONS: >-
        crowdsecurity/traefik
        crowdsecurity/http-cve
        crowdsecurity/base-http-scenarios
        crowdsecurity/sshd
      GID: "${GID-1000}"
      # Enroll with CrowdSec Console (optional, get key from app.crowdsec.net)
      # ENROLL_KEY: "your-enroll-key-here"
      # ENROLL_INSTANCE_NAME: "my-server"
      # ENROLL_TAGS: "docker traefik production"
    volumes:
      - ./crowdsec/acquis.yaml:/etc/crowdsec/acquis.yaml:ro
      - crowdsec-db:/var/lib/crowdsec/data
      - crowdsec-config:/etc/crowdsec
      - traefik-logs:/var/log/traefik:ro
    networks:
      - crowdsec
    expose:
      - "8080"    # LAPI (internal only)
      - "6060"    # Prometheus metrics (optional)

  # ─────────────────────────────────────────
  # Example Service (whoami)
  # ─────────────────────────────────────────
  whoami:
    image: traefik/whoami
    container_name: whoami
    restart: unless-stopped
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`whoami.example.com`)"
      - "traefik.http.routers.whoami.entrypoints=websecure"
      - "traefik.http.routers.whoami.tls.certresolver=letsencrypt"
      # Apply CrowdSec middleware
      - "traefik.http.routers.whoami.middlewares=crowdsec@file"
      - "traefik.http.services.whoami.loadbalancer.server.port=80"
```

**Key details:**
- Two networks: `proxy` (for Traefik ↔ services) and `crowdsec` (for Traefik ↔ CrowdSec).
- The `traefik-logs` volume is shared between Traefik (write) and CrowdSec (read-only).
- CrowdSec's LAPI port (8080) is only exposed internally, not published to the host.
- The `COLLECTIONS` environment variable auto-installs CrowdSec collections on first boot.

---

## Step 5: Enroll with CrowdSec Console

The CrowdSec Console provides a web dashboard for monitoring alerts, decisions, and machines.

1. **Create an account** at [app.crowdsec.net](https://app.crowdsec.net).
2. **Get your enrollment key** from the Console under "Security Engines" > "Add Security Engine".
3. **Add the enrollment key** to your `docker-compose.yml`:

```yaml
environment:
  ENROLL_KEY: "clxxxxxxxxxxxxxxxxxxxxxxxxxx"
  ENROLL_INSTANCE_NAME: "my-server"
  ENROLL_TAGS: "docker traefik production"
```

4. Alternatively, enroll after first boot:

```bash
docker exec crowdsec cscli console enroll <your-enroll-key>
```

5. **Accept the enrollment** in the CrowdSec Console web UI.

---

## Step 6: Register the Bouncer

The Traefik bouncer plugin needs an API key to communicate with CrowdSec's LAPI. This key must be generated **after** the CrowdSec container is running.

### First Boot (CrowdSec only)

```bash
cd ~/crowdsec-traefik

# Start CrowdSec first to initialize
docker compose up -d crowdsec

# Wait for it to initialize (~10 seconds)
sleep 10

# Register the bouncer and get the API key
docker exec crowdsec cscli bouncers add traefik-bouncer
```

This will output something like:

```
API key for 'traefik-bouncer':

   abcdef1234567890abcdef1234567890

Please keep this key since you will not be able to retrieve it!
```

### Set the API Key

Update the bouncer middleware configuration with the generated key.

**Option A: Directly in the file**

Edit `traefik/dynamic/middlewares.yml` and replace `${CROWDSEC_BOUNCER_API_KEY}` with the actual key:

```yaml
crowdsecLapiKey: "abcdef1234567890abcdef1234567890"
```

**Option B: Using an environment variable**

Create a `.env` file:

```bash
echo 'CROWDSEC_BOUNCER_API_KEY=abcdef1234567890abcdef1234567890' > .env
```

> **Important:** If using environment variables in Traefik's dynamic config, you may need to handle variable substitution through Traefik's `envFile` or by templating the config. The simplest approach is Option A (hardcode the key).

---

## Step 7: Deploy the Stack

```bash
cd ~/crowdsec-traefik

# Start the full stack
docker compose up -d

# Verify all containers are running
docker compose ps
```

Expected output:

```
NAME        STATUS          PORTS
crowdsec    Up (healthy)    8080/tcp, 6060/tcp
traefik     Up              0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp
whoami      Up              80/tcp
```

Check logs for errors:

```bash
# CrowdSec logs
docker compose logs crowdsec

# Traefik logs
docker compose logs traefik
```

---

## Step 8: Verify the Setup

### 8a. Check CrowdSec Status

```bash
# Check CrowdSec metrics
docker exec crowdsec cscli metrics

# List installed collections
docker exec crowdsec cscli collections list

# List installed parsers
docker exec crowdsec cscli parsers list

# List installed scenarios
docker exec crowdsec cscli scenarios list

# Check registered bouncers
docker exec crowdsec cscli bouncers list

# Check active decisions
docker exec crowdsec cscli decisions list
```

### 8b. Verify Log Parsing

```bash
# Check if CrowdSec is reading Traefik logs
docker exec crowdsec cscli metrics show acquisition

# You should see traefik log lines being read
```

### 8c. Test the Bouncer

Simulate a banned IP to verify the bouncer works:

```bash
# Add a test ban (use a non-critical IP)
docker exec crowdsec cscli decisions add --ip 1.2.3.4 --duration 1m --type ban

# Verify the decision exists
docker exec crowdsec cscli decisions list

# If you curl from that IP (or spoof), you should get a 403 Forbidden

# Remove the test ban
docker exec crowdsec cscli decisions delete --ip 1.2.3.4
```

### 8d. Simulate an Attack (Testing Only)

Use `nikto` or manual bad requests to trigger CrowdSec scenarios:

```bash
# Generate some bad requests to trigger scenarios
for i in $(seq 1 50); do
  curl -s -o /dev/null -w "%{http_code}\n" \
    "https://whoami.example.com/../../etc/passwd"
done

# Check if an alert was raised
docker exec crowdsec cscli alerts list

# Check if a decision (ban) was made
docker exec crowdsec cscli decisions list
```

---

## Step 9: Manage Collections and Scenarios

### Install Additional Collections

```bash
# List available collections from the hub
docker exec crowdsec cscli collections list -a

# Install a collection
docker exec crowdsec cscli collections install crowdsecurity/nginx
docker exec crowdsec cscli collections install crowdsecurity/linux

# Remove a collection
docker exec crowdsec cscli collections remove crowdsecurity/sshd
```

### Useful Collections for Traefik

| Collection | Purpose |
|------------|---------|
| `crowdsecurity/traefik` | Core Traefik log parser and scenarios |
| `crowdsecurity/base-http-scenarios` | HTTP attack detection (path traversal, scanners) |
| `crowdsecurity/http-cve` | Detection of CVE exploitation attempts |
| `crowdsecurity/sshd` | SSH brute force detection |
| `crowdsecurity/whitelist-good-actors` | Whitelist legitimate bots (Googlebot, etc.) |

### Update Hub Content

```bash
# Update all installed collections, parsers, and scenarios
docker exec crowdsec cscli hub update
docker exec crowdsec cscli hub upgrade
```

---

## Step 10: Maintenance and Operations

### View and Manage Decisions

```bash
# List all active bans
docker exec crowdsec cscli decisions list

# Manually ban an IP
docker exec crowdsec cscli decisions add --ip 203.0.113.50 --duration 24h --type ban --reason "Manual ban"

# Ban a subnet
docker exec crowdsec cscli decisions add --range 203.0.113.0/24 --duration 24h --type ban

# Unban an IP
docker exec crowdsec cscli decisions delete --ip 203.0.113.50

# Unban all
docker exec crowdsec cscli decisions delete --all
```

### Whitelist Trusted IPs

Create a whitelist file at `crowdsec/whitelist.yaml`:

```yaml
# crowdsec/whitelist.yaml
name: my-whitelist
description: "Whitelisted IPs"
whitelist:
  reason: "Trusted network"
  ip:
    - "10.0.0.0/8"
    - "192.168.0.0/16"
    - "172.16.0.0/12"
```

Mount it in the CrowdSec container:

```yaml
# Add to docker-compose.yml under crowdsec volumes
volumes:
  - ./crowdsec/whitelist.yaml:/etc/crowdsec/parsers/s02-enrich/my-whitelist.yaml:ro
```

### View Alerts

```bash
# List recent alerts
docker exec crowdsec cscli alerts list

# Get details on a specific alert
docker exec crowdsec cscli alerts inspect <alert-id>

# Delete old alerts
docker exec crowdsec cscli alerts delete --range 0.0.0.0/0
```

### Monitor with Prometheus (Optional)

CrowdSec exposes metrics on port 6060. Add to your Prometheus config:

```yaml
scrape_configs:
  - job_name: "crowdsec"
    static_configs:
      - targets: ["crowdsec:6060"]
```

### Backup and Restore

```bash
# Backup CrowdSec database
docker exec crowdsec cscli machines list
docker cp crowdsec:/var/lib/crowdsec/data/crowdsec.db ./backup/

# Backup configuration
docker cp crowdsec:/etc/crowdsec/ ./backup/crowdsec-config/
```

### Update CrowdSec

```bash
# Pull latest images
docker compose pull crowdsec

# Recreate with new image
docker compose up -d crowdsec

# Update hub content after upgrade
docker exec crowdsec cscli hub update
docker exec crowdsec cscli hub upgrade
```

---

## Troubleshooting

### CrowdSec Not Parsing Logs

**Symptom:** `cscli metrics` shows 0 lines parsed.

**Solutions:**
1. Verify the shared volume is mounted correctly:
   ```bash
   docker exec crowdsec ls -la /var/log/traefik/
   ```
2. Check `acquis.yaml` paths match the volume mount.
3. Ensure Traefik access logs are in JSON format:
   ```bash
   docker exec traefik cat /var/log/traefik/traefik-access.log | head -1
   ```
4. Restart CrowdSec after config changes:
   ```bash
   docker compose restart crowdsec
   ```

### Bouncer Returns 403 for All Requests

**Symptom:** All requests are blocked, not just malicious ones.

**Solutions:**
1. Check the bouncer API key is correct:
   ```bash
   docker exec crowdsec cscli bouncers list
   ```
2. Check Traefik logs for plugin errors:
   ```bash
   docker compose logs traefik | grep -i crowdsec
   ```
3. Verify CrowdSec LAPI is reachable from Traefik:
   ```bash
   docker exec traefik wget -qO- http://crowdsec:8080/health
   ```
4. Ensure the `crowdsec` network connects both containers.

### CrowdSec Container Crashes on Startup

**Symptom:** CrowdSec exits immediately.

**Solutions:**
1. Check logs:
   ```bash
   docker compose logs crowdsec
   ```
2. Common cause: corrupt database. Reset it:
   ```bash
   docker volume rm crowdsec-db
   docker compose up -d crowdsec
   ```
3. Check `acquis.yaml` for syntax errors:
   ```bash
   docker exec crowdsec cscli lapi status
   ```

### Traefik Plugin Not Loading

**Symptom:** Traefik logs show plugin download errors.

**Solutions:**
1. Ensure Traefik has internet access to download the plugin on first start.
2. Check the plugin version exists:
   ```
   https://plugins.traefik.io/plugins/6335346ca4caa9ddeffda116/crowdsec-bouncer-traefik-plugin
   ```
3. Verify the `experimental.plugins` section in `traefik.yml` has the correct `moduleName` and `version`.

### Real Client IP Not Detected

**Symptom:** CrowdSec sees internal Docker IPs instead of real client IPs.

**Solutions:**
1. Make sure `forwardedHeadersTrustedIPs` in the bouncer config includes your proxy/load balancer IPs.
2. If behind Cloudflare or another CDN, add their IP ranges to trusted IPs and configure Traefik's `forwardedHeaders`:
   ```yaml
   entryPoints:
     websecure:
       address: ":443"
       forwardedHeaders:
         trustedIPs:
           - "173.245.48.0/20"
           - "103.21.244.0/22"
           # ... other Cloudflare IPs
   ```
3. Use Traefik's `proxyProtocol` if your load balancer supports it.

---

## Security Hardening

### Production Checklist

- [ ] **Disable Traefik dashboard** or protect it with authentication middleware
- [ ] **Use HTTPS only** — redirect all HTTP to HTTPS
- [ ] **Restrict CrowdSec LAPI** — never expose port 8080 publicly
- [ ] **Use Docker secrets** for the bouncer API key instead of environment variables
- [ ] **Enable CrowdSec Console** enrollment for centralized monitoring
- [ ] **Set up log rotation** for Traefik access logs to prevent disk exhaustion
- [ ] **Subscribe to community blocklists** via the CrowdSec Console
- [ ] **Whitelist trusted IPs** (monitoring systems, internal networks)
- [ ] **Set `no-new-privileges`** security option on all containers
- [ ] **Mount Docker socket as read-only** (`:ro`)
- [ ] **Use specific image tags** instead of `latest` for reproducibility

### Log Rotation

Add logrotate configuration for Traefik access logs:

```bash
# /etc/logrotate.d/traefik
/path/to/crowdsec-traefik/logs/traefik-access.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
}
```

Or configure Traefik's built-in rotation:

```yaml
accessLog:
  filePath: /var/log/traefik/traefik-access.log
  bufferingSize: 100
  filters:
    statusCodes:
      - "200-599"
```

### Network Segmentation

Use separate Docker networks to isolate traffic:

```yaml
networks:
  proxy:        # Traefik <-> Services
  crowdsec:     # Traefik <-> CrowdSec (internal only)
  backend:      # Services <-> Databases (no external access)
```

---

## Quick Reference Commands

| Task | Command |
|------|---------|
| Start stack | `docker compose up -d` |
| Stop stack | `docker compose down` |
| View CrowdSec metrics | `docker exec crowdsec cscli metrics` |
| List bans | `docker exec crowdsec cscli decisions list` |
| Ban an IP | `docker exec crowdsec cscli decisions add --ip X.X.X.X --duration 24h --type ban` |
| Unban an IP | `docker exec crowdsec cscli decisions delete --ip X.X.X.X` |
| List alerts | `docker exec crowdsec cscli alerts list` |
| List bouncers | `docker exec crowdsec cscli bouncers list` |
| Update hub | `docker exec crowdsec cscli hub update && docker exec crowdsec cscli hub upgrade` |
| Check LAPI status | `docker exec crowdsec cscli lapi status` |
| View CrowdSec logs | `docker compose logs -f crowdsec` |
| View Traefik logs | `docker compose logs -f traefik` |
