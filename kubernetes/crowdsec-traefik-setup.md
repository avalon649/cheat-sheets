# CrowdSec + Traefik Setup Guide (ArgoCD GitOps)

**Date:** 2026-02-15
**Environment:** K3s-ProD / Traefik v3.4.3 (Helm) / ArgoCD
**Repo:** `https://github.com/avalon649/argocd-prod`

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Prerequisites](#prerequisites)
3. [Step 1: Update Traefik Helm Values](#step-1-update-traefik-helm-values)
4. [Step 2: CrowdSec LAPI Manifests (ArgoCD)](#step-2-crowdsec-lapi-manifests-argocd)
5. [Step 3: CrowdSec Bouncer Middleware (ArgoCD)](#step-3-crowdsec-bouncer-middleware-argocd)
6. [Step 4: ArgoCD Application Manifests](#step-4-argocd-application-manifests)
7. [Step 5: Deploy](#step-5-deploy)
8. [Step 6: Register the Bouncer](#step-6-register-the-bouncer)
9. [Step 7: Apply Middleware to Ingresses](#step-7-apply-middleware-to-ingresses)
10. [Step 8: Verify](#step-8-verify)
11. [Step 9: CrowdSec Console Enrollment](#step-9-crowdsec-console-enrollment)
12. [Maintenance Commands](#maintenance-commands)

---

## Architecture Overview

```
                        Internet
                            |
                      [ MetalLB ]
                       10.0.21.80
                            |
                    +--------------+
                    |   Traefik    |  <-- Bouncer Plugin checks every request
                    |  (websecure) |      against CrowdSec LAPI decisions
                    +------+-------+
                           |
              +------------+------------+
              |            |            |
         [ ArgoCD ]  [ Vaultwarden ]  [ ... all services ]

                    Traefik Pod
              +---------------------+
              | traefik container   | --> writes /var/log/traefik/access.log
              | crowdsec-agent      | --> reads access.log, parses, sends to LAPI
              +---------------------+
                           |
                    +------+-------+
                    | CrowdSec     |
                    | LAPI         |  <-- Stores decisions, community blocklists
                    | (crowdsec ns)|
                    +--------------+
```

**Components:**

| Component | Where | Managed By |
|-----------|-------|------------|
| CrowdSec LAPI | `crowdsec` namespace | ArgoCD (plain YAML) |
| CrowdSec Agent | Sidecar in Traefik pod | Traefik Helm values |
| Bouncer Plugin | Traefik plugin | Traefik Helm values |
| Bouncer Middleware | `traefik` namespace | ArgoCD (plain YAML) |

---

## Prerequisites

### 1. Create NFS directories on TrueNAS (10.0.21.20)

```bash
ssh root@10.0.21.20
mkdir -p /mnt/storage0/k3s/crowdsec/data
mkdir -p /mnt/storage0/k3s/crowdsec/config
```

### 2. Ensure Traefik Helm repo is available

```bash
helm repo add traefik https://traefik.github.io/charts
helm repo update
```

### 3. Check current Traefik Helm values

```bash
helm get values traefik -n traefik
```

---

## Step 1: Update Traefik Helm Values

Traefik is Helm-managed (not via ArgoCD), so these changes are applied via `helm upgrade`.

### Create the values override file

Save this as `/home/avalon/traefik-crowdsec-values.yaml`:

```yaml
# /home/avalon/traefik-crowdsec-values.yaml
# CrowdSec integration values for Traefik Helm chart

# -----------------------------------------------
# 1. Enable Access Logs (required for CrowdSec)
# -----------------------------------------------
additionalArguments:
  - "--accesslog=true"
  - "--accesslog.filepath=/var/log/traefik/access.log"
  - "--accesslog.format=json"
  - "--accesslog.fields.headers.defaultmode=keep"
  # CrowdSec Bouncer Plugin
  - "--experimental.plugins.crowdsec.modulename=github.com/maxlerebourg/crowdsec-bouncer-traefik-plugin"
  - "--experimental.plugins.crowdsec.version=v1.3.5"

# -----------------------------------------------
# 2. Shared log volume for Traefik + Agent sidecar
# -----------------------------------------------
additionalVolumeMounts:
  - name: traefik-logs
    mountPath: /var/log/traefik

# -----------------------------------------------
# 3. CrowdSec Agent sidecar container
# -----------------------------------------------
deployment:
  additionalVolumes:
    - name: traefik-logs
      emptyDir: {}
    - name: crowdsec-agent-config
      configMap:
        name: crowdsec-agent-config

  additionalContainers:
    - name: crowdsec-agent
      image: crowdsecurity/crowdsec:v1.6.4
      env:
        - name: COLLECTIONS
          value: "crowdsecurity/traefik crowdsecurity/http-cve crowdsecurity/base-http-scenarios"
        - name: PARSERS
          value: "crowdsecurity/traefik-logs"
        - name: DISABLE_LOCAL_API
          value: "true"
        - name: AGENT_USERNAME
          value: "traefik-agent"
        - name: AGENT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: crowdsec-agent-credentials
              key: agent-password
        - name: LOCAL_API_URL
          value: "http://crowdsec-lapi-svc.crowdsec.svc.cluster.local:8080"
      volumeMounts:
        - name: traefik-logs
          mountPath: /var/log/traefik
          readOnly: true
        - name: crowdsec-agent-config
          mountPath: /etc/crowdsec/acquis.d
      resources:
        limits:
          memory: 256Mi
          cpu: 200m
        requests:
          memory: 128Mi
          cpu: 100m
```

### Create the agent acquisition ConfigMap

This must exist in the `traefik` namespace before upgrading:

```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: crowdsec-agent-config
  namespace: traefik
data:
  acquis.yaml: |
    filenames:
      - /var/log/traefik/access.log
    labels:
      type: traefik
EOF
```

### Create the agent credentials secret

Generate a password and create the secret (used later when registering the agent with LAPI):

```bash
AGENT_PASSWORD=$(openssl rand -hex 16)
echo "Save this agent password: $AGENT_PASSWORD"

kubectl create secret generic crowdsec-agent-credentials \
  -n traefik \
  --from-literal=agent-password="$AGENT_PASSWORD"
```

### Apply the Helm upgrade

**Important:** Do NOT run this until Step 2 (CrowdSec LAPI) is deployed first.

```bash
helm upgrade traefik traefik/traefik \
  -n traefik \
  -f /home/avalon/traefik-crowdsec-values.yaml \
  --reuse-values
```

---

## Step 2: CrowdSec LAPI Manifests (ArgoCD)

Create these files in your ArgoCD repo under `crowdsec/`.

### Directory structure

```
argocd-prod/
  crowdsec/
    ns.yml
    lapi-deploy.yml
    lapi-svc.yml
    lapi-configmap.yml
```

---

### `crowdsec/ns.yml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: crowdsec
  labels:
    name: crowdsec
```

---

### `crowdsec/lapi-configmap.yml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: crowdsec-lapi-config
  namespace: crowdsec
data:
  # Profiles define what happens when a scenario triggers
  profiles.yaml: |
    name: default_ip_remediation
    filters:
      - Alert.Remediation == true && Alert.GetScope() == "Ip"
    decisions:
      - type: ban
        duration: 4h
    on_success: break

  # Local API server configuration overrides
  config.yaml.local: |
    db_config:
      type: sqlite
      db_path: /var/lib/crowdsec/data/crowdsec.db
    api:
      server:
        listen_uri: 0.0.0.0:8080
```

---

### `crowdsec/lapi-deploy.yml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: crowdsec-lapi
  namespace: crowdsec
  labels:
    app: crowdsec-lapi
spec:
  selector:
    matchLabels:
      app: crowdsec-lapi
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: crowdsec-lapi
    spec:
      containers:
      - name: crowdsec-lapi
        image: crowdsecurity/crowdsec:v1.6.4
        env:
        - name: DISABLE_AGENT
          value: "true"
        - name: COLLECTIONS
          value: "crowdsecurity/traefik crowdsecurity/http-cve crowdsecurity/base-http-scenarios crowdsecurity/linux"
        - name: PARSERS
          value: "crowdsecurity/traefik-logs"
        ports:
        - containerPort: 8080
          name: lapi
          protocol: TCP
        - containerPort: 6060
          name: metrics
          protocol: TCP
        resources:
          limits:
            memory: 512Mi
            cpu: 500m
          requests:
            memory: 256Mi
            cpu: 100m
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 15
          timeoutSeconds: 5
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 10
          timeoutSeconds: 5
        volumeMounts:
        - name: crowdsec-data
          mountPath: /var/lib/crowdsec/data
        - name: crowdsec-config
          mountPath: /etc/crowdsec
        - name: crowdsec-lapi-config
          mountPath: /etc/crowdsec/profiles.yaml
          subPath: profiles.yaml
        - name: crowdsec-lapi-config
          mountPath: /etc/crowdsec/config.yaml.local
          subPath: config.yaml.local
      volumes:
      - name: crowdsec-data
        nfs:
          path: /mnt/storage0/k3s/crowdsec/data
          server: 10.0.21.20
          readOnly: no
      - name: crowdsec-config
        nfs:
          path: /mnt/storage0/k3s/crowdsec/config
          server: 10.0.21.20
          readOnly: no
      - name: crowdsec-lapi-config
        configMap:
          name: crowdsec-lapi-config
```

---

### `crowdsec/lapi-svc.yml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: crowdsec-lapi-svc
  namespace: crowdsec
spec:
  selector:
    app: crowdsec-lapi
  type: ClusterIP
  ports:
  - name: lapi
    port: 8080
    targetPort: 8080
    protocol: TCP
  - name: metrics
    port: 6060
    targetPort: 6060
    protocol: TCP
```

---

## Step 3: CrowdSec Bouncer Middleware (ArgoCD)

Create these files under `crowdsec-bouncer/` in your ArgoCD repo.

### Directory structure

```
argocd-prod/
  crowdsec-bouncer/
    middleware.yml
```

---

### `crowdsec-bouncer/middleware.yml`

The bouncer API key is populated in [Step 6](#step-6-register-the-bouncer) after the LAPI is running.

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: crowdsec-bouncer
  namespace: traefik
spec:
  plugin:
    crowdsec:
      crowdsecLapiScheme: http
      crowdsecLapiHost: crowdsec-lapi-svc.crowdsec.svc.cluster.local:8080
      crowdsecLapiKey: REPLACE_WITH_BOUNCER_API_KEY
      updateIntervalSeconds: 15
      defaultDecisionSeconds: 60
      crowdsecMode: live
      forwardedHeadersTrustedIPs:
        - 10.0.0.0/8
        - 172.16.0.0/12
      clientTrustedIPs:
        - 10.0.20.0/24
        - 10.0.21.0/24
```

---

## Step 4: ArgoCD Application Manifests

Apply these directly with `kubectl` to register the apps in ArgoCD.

### CrowdSec LAPI Application

```bash
kubectl apply -f - <<'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: crowdsec
  namespace: argocd
spec:
  destination:
    namespace: crowdsec
    server: https://kubernetes.default.svc
  project: default
  source:
    path: crowdsec
    repoURL: https://github.com/avalon649/argocd-prod
    targetRevision: HEAD
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF
```

### CrowdSec Bouncer Middleware Application

```bash
kubectl apply -f - <<'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: crowdsec-bouncer
  namespace: argocd
spec:
  destination:
    namespace: traefik
    server: https://kubernetes.default.svc
  project: default
  source:
    path: crowdsec-bouncer
    repoURL: https://github.com/avalon649/argocd-prod
    targetRevision: HEAD
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF
```

---

## Step 5: Deploy

Follow this order carefully.

### 5.1 Push manifests to GitHub

```bash
cd ~/argocd-prod   # or wherever your local clone is

# Create the directories
mkdir -p crowdsec
mkdir -p crowdsec-bouncer

# Copy/create the manifest files listed above into each directory, then:
git add crowdsec/ crowdsec-bouncer/
git commit -m "Add CrowdSec LAPI and bouncer middleware manifests"
git push origin main
```

### 5.2 Register ArgoCD applications

```bash
# Apply both ArgoCD Application manifests from Step 4
# ArgoCD will auto-sync since syncPolicy.automated is set
```

### 5.3 Wait for CrowdSec LAPI to be ready

```bash
# Watch the LAPI pod come up
kubectl get pods -n crowdsec -w

# Verify LAPI health
kubectl logs -n crowdsec -l app=crowdsec-lapi --tail=50
```

### 5.4 Register the agent and bouncer (Step 6 below)

### 5.5 Upgrade Traefik Helm

Only after LAPI is running and the agent/bouncer are registered:

```bash
helm upgrade traefik traefik/traefik \
  -n traefik \
  -f /home/avalon/traefik-crowdsec-values.yaml \
  --reuse-values
```

---

## Step 6: Register the Bouncer

Once the CrowdSec LAPI pod is running:

### 6.1 Get the LAPI pod name

```bash
LAPI_POD=$(kubectl get pods -n crowdsec -l app=crowdsec-lapi -o jsonpath='{.items[0].metadata.name}')
echo $LAPI_POD
```

### 6.2 Register the bouncer

```bash
kubectl exec -n crowdsec $LAPI_POD -- cscli bouncers add traefik-bouncer
```

This outputs an API key like:

```
API key for 'traefik-bouncer':

   abc123def456...

Please keep this key since you will not be able to retrieve it!
```

**Save this key.**

### 6.3 Register the agent

```bash
# Use the same AGENT_PASSWORD you created earlier
kubectl exec -n crowdsec $LAPI_POD -- \
  cscli machines add traefik-agent --password "$AGENT_PASSWORD" --url http://localhost:8080
```

### 6.4 Update the bouncer middleware with the API key

Edit `crowdsec-bouncer/middleware.yml` in your repo, replacing `REPLACE_WITH_BOUNCER_API_KEY` with the actual key:

```yaml
      crowdsecLapiKey: abc123def456...    # <-- paste the real key here
```

Then push:

```bash
cd ~/argocd-prod
git add crowdsec-bouncer/middleware.yml
git commit -m "Add CrowdSec bouncer API key"
git push origin main
```

ArgoCD will auto-sync the middleware.

### 6.5 (Alternative) Store the API key as a Secret

If you prefer not to have the API key in Git, use a Kubernetes secret instead.

Create the secret manually:

```bash
kubectl create secret generic crowdsec-bouncer-key \
  -n traefik \
  --from-literal=api-key="abc123def456..."
```

Then modify the middleware to reference it via Traefik environment variables. This requires adding to the Traefik Helm values:

```yaml
# In traefik-crowdsec-values.yaml, add under env:
env:
  - name: CROWDSEC_BOUNCER_API_KEY
    valueFrom:
      secretKeyRef:
        name: crowdsec-bouncer-key
        key: api-key
```

And in the middleware, use the environment variable syntax supported by the plugin.

---

## Step 7: Apply Middleware to Ingresses

You have two options to attach the CrowdSec bouncer to your services.

### Option A: Default middleware on the entrypoint (all traffic)

Add this to your Traefik Helm values so **every** request on `websecure` goes through CrowdSec:

```yaml
# Add to additionalArguments in traefik-crowdsec-values.yaml
additionalArguments:
  - "--entryPoints.websecure.http.middlewares=traefik-crowdsec-bouncer@kubernetescrd"
```

Then upgrade:

```bash
helm upgrade traefik traefik/traefik \
  -n traefik \
  -f /home/avalon/traefik-crowdsec-values.yaml \
  --reuse-values
```

This is the **recommended** approach -- protects everything without modifying each app.

### Option B: Per-ingress annotation

If you want to apply CrowdSec only to specific services, add the annotation to individual Ingress manifests in your ArgoCD repo.

Example for `uptime-kuma/ingress.yml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: uptime-kuma-ingress
  namespace: uptime-kuma
  annotations:
    traefik.ingress.kubernetes.io/router.middlewares: traefik-crowdsec-bouncer@kubernetescrd
spec:
  rules:
  - host: kuma.local.kore-zone.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: uptime-kuma-svc
            port:
              number: 3001
```

The annotation format is `<namespace>-<middleware-name>@kubernetescrd`.

To apply to all your apps at once, update these ingress files:

| App | File to update |
|-----|---------------|
| uptime-kuma | `uptime-kuma/ingress.yml` |
| freshrss | `freshrss/ingress.yml` |
| n8n | `n8n/ingress.yml` |
| nextcloud | `nextcloud/ingress.yml` |
| searxng | `searxng/ingress.yml` |
| vaultwarden | `vaultwarden/ingress.yml` |
| code-server | `code-server/ingress.yml` |
| pgadmin | `pgadmin/ingress.yml` |
| docs | `docs/ingress.yml` |
| homepage | `homepage/ingress.yml` |
| it-tools | `it-tools/ingress.yml` |
| cyberchef | `cyberchef/ingress.yml` |

---

## Step 8: Verify

### 8.1 Check all pods are running

```bash
kubectl get pods -n crowdsec
kubectl get pods -n traefik
```

Expected output for Traefik pod:

```
NAME                       READY   STATUS    RESTARTS   AGE
traefik-xxxx-xxxxx         2/2     Running   0          5m    # 2/2 = traefik + crowdsec-agent
```

### 8.2 Check CrowdSec LAPI health

```bash
LAPI_POD=$(kubectl get pods -n crowdsec -l app=crowdsec-lapi -o jsonpath='{.items[0].metadata.name}')

# Check registered machines (agents)
kubectl exec -n crowdsec $LAPI_POD -- cscli machines list

# Check registered bouncers
kubectl exec -n crowdsec $LAPI_POD -- cscli bouncers list

# Check installed collections
kubectl exec -n crowdsec $LAPI_POD -- cscli collections list

# Check metrics (parsed logs, scenarios triggered)
kubectl exec -n crowdsec $LAPI_POD -- cscli metrics
```

### 8.3 Check the agent is parsing logs

```bash
# Check agent container logs in the Traefik pod
TRAEFIK_POD=$(kubectl get pods -n traefik -l app.kubernetes.io/name=traefik -o jsonpath='{.items[0].metadata.name}')

kubectl logs -n traefik $TRAEFIK_POD -c crowdsec-agent --tail=20
```

### 8.4 Check the middleware is active

```bash
kubectl get middlewares -n traefik
```

Expected:

```
NAME                         AGE
crowdsec-bouncer             5m
traefik-dashboard-basicauth  260d
```

### 8.5 Check active decisions (bans)

```bash
kubectl exec -n crowdsec $LAPI_POD -- cscli decisions list
```

### 8.6 Test with a simulated scan

From an external machine or different network:

```bash
# Simulate a basic directory traversal attack
for i in $(seq 1 50); do
  curl -s -o /dev/null -w "%{http_code}\n" \
    "https://kuma.local.kore-zone.com/../../etc/passwd"
done

# After a few seconds, check if the IP was banned
kubectl exec -n crowdsec $LAPI_POD -- cscli decisions list
```

---

## Step 9: CrowdSec Console Enrollment

The CrowdSec Console at [app.crowdsec.net](https://app.crowdsec.net) gives you:
- Web dashboard for alerts and decisions
- Community blocklists (shared threat intelligence)
- Multi-instance management

### 9.1 Create a free account

Sign up at https://app.crowdsec.net

### 9.2 Get your enrollment key

After signing in, go to **Security Engines** > **Engines** > **Add**. Copy the enrollment key.

### 9.3 Enroll your instance

```bash
kubectl exec -n crowdsec $LAPI_POD -- \
  cscli console enroll <YOUR_ENROLLMENT_KEY>
```

### 9.4 Subscribe to community blocklists

In the CrowdSec Console, go to **Blocklists** and subscribe to lists like:
- CrowdSec Community Blocklist (free)
- Tor exit nodes
- Known botnets

---

## Maintenance Commands

### View alerts

```bash
kubectl exec -n crowdsec $LAPI_POD -- cscli alerts list
```

### Ban an IP manually

```bash
kubectl exec -n crowdsec $LAPI_POD -- \
  cscli decisions add --ip 1.2.3.4 --duration 24h --reason "manual ban"
```

### Unban an IP

```bash
kubectl exec -n crowdsec $LAPI_POD -- cscli decisions delete --ip 1.2.3.4
```

### View what scenarios triggered

```bash
kubectl exec -n crowdsec $LAPI_POD -- cscli alerts list --since 1h
```

### Update hub (parsers, scenarios, collections)

```bash
kubectl exec -n crowdsec $LAPI_POD -- cscli hub update
kubectl exec -n crowdsec $LAPI_POD -- cscli hub upgrade
```

### View metrics (parsed logs, active decisions)

```bash
kubectl exec -n crowdsec $LAPI_POD -- cscli metrics
```

### Whitelist an IP

```bash
kubectl exec -n crowdsec $LAPI_POD -- \
  cscli decisions delete --ip 10.0.20.1
# For permanent whitelist, add to /etc/crowdsec/parsers/s02-enrich/whitelist.yaml
```

### Check agent connectivity

```bash
kubectl exec -n crowdsec $LAPI_POD -- cscli machines list
# Status should show "validated" and recent last_heartbeat
```

---

## File Summary

### Files to add to `argocd-prod` repo

```
argocd-prod/
├── crowdsec/
│   ├── ns.yml                  # Namespace
│   ├── lapi-deploy.yml         # LAPI Deployment with NFS volumes
│   ├── lapi-svc.yml            # ClusterIP service for LAPI
│   └── lapi-configmap.yml      # Profiles and config overrides
├── crowdsec-bouncer/
│   └── middleware.yml          # Traefik Middleware CRD for bouncer plugin
```

### Files on local machine

```
/home/avalon/
├── traefik-crowdsec-values.yaml  # Helm values override for Traefik
```

### Secrets created manually (not in Git)

| Secret | Namespace | Purpose |
|--------|-----------|---------|
| `crowdsec-agent-credentials` | `traefik` | Agent password for LAPI auth |
| `crowdsec-bouncer-key` | `traefik` | Bouncer API key (if using Option 6.5) |

### ArgoCD Applications to register

| App Name | Path | Namespace |
|----------|------|-----------|
| `crowdsec` | `crowdsec/` | `crowdsec` |
| `crowdsec-bouncer` | `crowdsec-bouncer/` | `traefik` |

---

## Deployment Order Checklist

```
[ ] 1. Create NFS directories on TrueNAS (10.0.21.20)
[ ] 2. Push crowdsec/ and crowdsec-bouncer/ manifests to GitHub
[ ] 3. Apply ArgoCD Application manifests (Step 4)
[ ] 4. Wait for CrowdSec LAPI pod to be Ready
[ ] 5. Create crowdsec-agent-credentials secret in traefik namespace
[ ] 6. Register agent:  cscli machines add traefik-agent ...
[ ] 7. Register bouncer: cscli bouncers add traefik-bouncer
[ ] 8. Update crowdsec-bouncer/middleware.yml with bouncer API key, push to Git
[ ] 9. Create crowdsec-agent-config ConfigMap in traefik namespace
[ ] 10. Run helm upgrade with traefik-crowdsec-values.yaml
[ ] 11. Verify Traefik pod shows 2/2 containers
[ ] 12. Verify cscli metrics shows parsed logs
[ ] 13. Enroll in CrowdSec Console (optional)
[ ] 14. Test with simulated attack
```

---

*Generated by Claude Code - CrowdSec + Traefik Integration Guide*
