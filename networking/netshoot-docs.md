# Netshoot — Docker & Kubernetes Network Troubleshooting

## Overview

Netshoot (`nicolaka/netshoot`) is a Docker and Kubernetes network troubleshooting container that bundles ~50+ powerful diagnostic tools. It leverages **network namespaces** — isolated networking environments within containers — allowing you to enter a different container's network namespace and perform troubleshooting with tools that aren't even installed on that container.

**Image:** `nicolaka/netshoot`
**License:** Apache 2.0
**Repository:** https://github.com/nicolaka/netshoot

---

## Included Tools

### Packages

| Category | Tools |
|---|---|
| DNS | bind-tools, drill |
| Traffic Capture | tcpdump, tshark, termshark, ngrep |
| Traffic Monitoring | iftop, iptraf-ng |
| Connectivity | curl, netcat-openbsd, socat, nmap, nmap-nping |
| Performance | iperf, iperf3, fortio |
| Routing & Firewall | iproute2, bridge-utils, iptables, nftables, conntrack-tools |
| Debugging | strace, ltrace, ethtool |
| HTTP & Web | apache2-utils, curl, openssl |
| Scripting | bash, zsh, jq, vim |
| Other | dhcping, scapy, tcptraceroute, mtr |

### Additional Binaries

| Tool | Purpose |
|---|---|
| ctop | Container metrics monitoring |
| calicoctl | Calico networking management |
| termshark | TUI packet analysis (Wireshark-like) |
| grpcurl | gRPC endpoint testing |
| fortio | Load testing |

---

## Docker Usage

### Attach to a Container's Network Namespace

The most common use — debug from the perspective of another running container:

```bash
docker run -it --rm --net container:<container_name> nicolaka/netshoot
```

### Attach to the Host's Network Namespace

Troubleshoot Docker host networking directly:

```bash
docker run -it --rm --net host nicolaka/netshoot
```

### Docker Compose Sidecar (e.g. Packet Capture)

```yaml
version: "3.6"
services:
  tcpdump:
    image: nicolaka/netshoot
    depends_on:
      - nginx
    command: tcpdump -i eth0 -w /data/nginx.pcap
    network_mode: service:nginx
    volumes:
      - $PWD/data:/data

  nginx:
    image: nginx:alpine
    ports:
      - 80:80
```

---

## Tool Usage Examples (Docker)

### iperf — Network Performance Testing

```bash
# Create a test network
docker network create -d bridge perf-test

# Start iperf server
docker run -d --rm --net perf-test --name perf-test-a nicolaka/netshoot iperf -s -p 9999

# Run iperf client against the server
docker run -it --rm --net perf-test --name perf-test-b nicolaka/netshoot iperf -c perf-test-a -p 9999
```

### tcpdump — Packet Capture

```bash
# Capture on a specific container's network
docker run -it --rm --net container:<target_container> nicolaka/netshoot tcpdump -i eth0 port 9999 -c 1 -Xvv
```

### netstat — Check Listening Ports

```bash
docker run -it --rm --net container:<target_container> nicolaka/netshoot netstat -tulpn
```

### nmap — Port Scanning

```bash
docker run -it --rm --privileged nicolaka/netshoot nmap -p 12376-12390 -dd 172.31.24.25
```

### iftop — Live Bandwidth Monitoring

```bash
docker run -it --rm --net container:<target_container> nicolaka/netshoot iftop -i eth0
```

### drill — DNS Resolution Debugging

```bash
docker run -it --rm --net container:<target_container> nicolaka/netshoot drill -V 5 <hostname>
```

### netcat — TCP/UDP Connectivity Testing

```bash
# Create a test network
docker network create -d bridge my-br

# Start a listener
docker run -d --rm --net my-br --name service-a nicolaka/netshoot nc -l 8080

# Test connection from another container
docker run -it --rm --net my-br --name service-b nicolaka/netshoot nc -vz service-a 8080
```

### iproute2 — Routing and Neighbor Tables

```bash
docker run -it --rm --net host nicolaka/netshoot
# Then inside:
ip route show
ip neigh show
```

### ctop — Live Container Metrics

```bash
docker run -it --rm -v /var/run/docker.sock:/var/run/docker.sock nicolaka/netshoot ctop
```

### termshark — TUI Packet Analyzer

```bash
# Live capture
docker run --rm --cap-add=NET_ADMIN --cap-add=NET_RAW -it nicolaka/netshoot termshark -i eth0 icmp

# Read from pcap file
docker run --rm --cap-add=NET_ADMIN --cap-add=NET_RAW \
  -v /tmp/capture.pcap:/tmp/capture.pcap \
  -it nicolaka/netshoot termshark -r /tmp/capture.pcap
```

### nsenter — Enter Docker Network Namespaces Directly

```bash
docker run -it --rm \
  -v /var/run/docker/netns:/var/run/docker/netns \
  --privileged=true nicolaka/netshoot

# Then inside:
cd /var/run/docker/netns/
nsenter --net=/var/run/docker/netns/<namespace_id> sh
```

### grpcurl — gRPC Endpoint Testing

```bash
# With TLS
grpcurl grpc.server.com:443 my.custom.server.Service/Method

# Without TLS
grpcurl -plaintext grpc.server.com:80 my.custom.server.Service/Method
```

### fortio — Load Testing

```bash
fortio load http://www.google.com
```

---

## Kubernetes Usage

### Ephemeral Debug Container (Recommended)

Attach netshoot directly to a running pod's network namespace:

```bash
kubectl debug <pod-name> -n <namespace> -it \
  --image=nicolaka/netshoot \
  --target=<container-name> \
  -- bash
```

Example with Prometheus:

```bash
kubectl debug prometheus-prometheus-prometheus-0 -n monitoring -it \
  --image=nicolaka/netshoot \
  --target=prometheus \
  -- tcpdump -i eth0 port 9090 -Xvv
```

### Throwaway Debug Pod

```bash
kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot
```

### Pod on Host Network Namespace

```bash
kubectl run tmp-shell --rm -i --tty \
  --overrides='{"spec": {"hostNetwork": true}}' \
  --image nicolaka/netshoot
```

### Sidecar Container

Add netshoot as a sidecar in your pod spec:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: app
      image: my-app:latest
    - name: netshoot
      image: nicolaka/netshoot
      command: ["sleep", "infinity"]
      securityContext:
        capabilities:
          add: ["NET_ADMIN", "NET_RAW"]
```

Then exec into it:

```bash
kubectl exec -it my-app -c netshoot -- bash
```

### Debugging pods with non root user capabilities

```bash
kubectl debug -it grafana-bbf56c487-49s5l -n monitoring \
  --copy-to=grafana-debug \
  --image=nicolaka/netshoot \
  --custom=/tmp/debug-profile.json \
  --share-processes \
  -- bash
```
### With /tmp/debug-profile.json containing:

```json  
{
  "securityContext": {
    "runAsUser": 0,
    "runAsNonRoot": false,
    "capabilities": {
      "add": ["NET_RAW", "NET_ADMIN"]
    }
  }
}
```
---

## Common Troubleshooting Scenarios

### 1. Container Can't Reach Another Container

```bash
# Attach to source container's network
docker run -it --rm --net container:<source> nicolaka/netshoot

# Test DNS resolution
drill <target_hostname>

# Test TCP connectivity
nc -vz <target_hostname> <port>

# Trace the route
mtr <target_ip>
```

### 2. Debug Listening Ports

```bash
docker run -it --rm --net container:<target> nicolaka/netshoot

# Check what's listening
netstat -tulpn
ss -tlnp
```

### 3. Capture Traffic for Analysis

```bash
# Capture to file
docker run -it --rm --net container:<target> \
  -v $PWD:/data nicolaka/netshoot \
  tcpdump -i eth0 -w /data/capture.pcap

# Then analyze with termshark
docker run --rm --cap-add=NET_ADMIN --cap-add=NET_RAW \
  -v $PWD/capture.pcap:/tmp/capture.pcap \
  -it nicolaka/netshoot termshark -r /tmp/capture.pcap
```

### 4. Test Bandwidth Between Containers

```bash
# Server side
docker run -d --rm --net container:<container_a> --name perf-server \
  nicolaka/netshoot iperf -s -p 5001

# Client side
docker run -it --rm --net container:<container_b> \
  nicolaka/netshoot iperf -c <container_a_ip> -p 5001
```

### 5. Inspect Network Configuration

```bash
docker run -it --rm --net container:<target> nicolaka/netshoot

# View interfaces
ifconfig
ip addr show

# View routes
ip route show

# View ARP/neighbor table
ip neigh show

# View iptables rules
iptables -L -n -v
```

---

## Key Capabilities Reference

| Capability | Required For |
|---|---|
| `NET_RAW` | tcpdump, nmap, ping, nping |
| `NET_ADMIN` | iptables, ip route, termshark |
| `SYS_PTRACE` | strace, ltrace |
| `--privileged` | Full namespace access, nsenter |

### Docker Capability Flags

```bash
# Add specific capabilities
docker run --cap-add=NET_ADMIN --cap-add=NET_RAW -it nicolaka/netshoot

# Full privileged mode (use with caution)
docker run --privileged -it nicolaka/netshoot
```

---

## Quick Reference

| Task | Command |
|---|---|
| Enter container network | `docker run -it --rm --net container:<name> nicolaka/netshoot` |
| Enter host network | `docker run -it --rm --net host nicolaka/netshoot` |
| Capture packets | `tcpdump -i eth0 port <port> -Xvv` |
| Check open ports | `netstat -tulpn` or `ss -tlnp` |
| Test connectivity | `nc -vz <host> <port>` |
| DNS lookup | `drill <hostname>` |
| Trace route | `mtr <host>` |
| Bandwidth test | `iperf -c <server> -p <port>` |
| Monitor bandwidth | `iftop -i eth0` |
| Port scan | `nmap -p <range> <target>` |
| Container metrics | `ctop` (needs docker socket) |
| Packet analysis TUI | `termshark -i eth0` |
| Load test | `fortio load <url>` |
| K8s ephemeral debug | `kubectl debug <pod> -n <ns> -it --image=nicolaka/netshoot --target=<container>` |
| K8s throwaway pod | `kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot` |
