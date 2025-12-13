# Kubernetes on Ubuntu Cheat Sheet

This guide provides the essential commands and YAML examples to set up, deploy, and maintain a Kubernetes cluster on Ubuntu using `kubeadm`.

---

## 1. Installation and Setup

These commands should be run on all nodes (master and workers) that will be part of the cluster.

### Step 1: Prepare the System

```bash
# Update package list and install required packages
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# Disable swap (Kubernetes requirement)
sudo swapoff -a
# And make it permanent by commenting it out in /etc/fstab
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Load required kernel modules
sudo tee /etc/modules-load.d/k8s.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

# Configure sysctl for Kubernetes networking
sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```

### Step 2: Install Containerd (Container Runtime)

```bash
# Install containerd
sudo apt-get install -y containerd

# Configure containerd to use systemd cgroup driver
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

# Restart containerd to apply changes
sudo systemctl restart containerd
```

### Step 3: Install Kubernetes Tools (`kubeadm`, `kubelet`, `kubectl`)

```bash
# Add the Kubernetes repository GPG key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the Kubernetes repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update package list and install Kubernetes components
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

# Pin the versions to prevent unintended upgrades
sudo apt-mark hold kubelet kubeadm kubectl
```

---

## 2. Cluster Initialization

### Step 1: Initialize the Master Node

**Run this command only on your master node.** Replace `<MASTER_IP>` with your master node's private IP address.

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=<MASTER_IP>
```

After it finishes, it will output a `kubeadm join` command. **Save this command!** You will need it to add worker nodes.

### Step 2: Configure `kubectl` for the Master Node

```bash
# Create the .kube directory
mkdir -p $HOME/.kube

# Copy the admin configuration to your user's kube config
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Step 3: Install a Pod Network Add-on (Calico)

A CNI (Container Network Interface) is required for pods to communicate.

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
```

### Step 4: Join Worker Nodes

Run the `kubeadm join` command that you saved from the `kubeadm init` output on each worker node. It will look something like this:

```bash
sudo kubeadm join <MASTER_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```

---

## 3. Cluster Maintenance and Management (`kubectl`)

These commands are your daily drivers for interacting with the cluster.

| Command | Description |
|---|---|
| `kubectl get nodes` | List all nodes in the cluster and their status. |
| `kubectl get pods -A` | List all pods in all namespaces. |
| `kubectl get pods -n [namespace]`| List pods in a specific namespace. |
| `kubectl describe node [node-name]` | Show detailed information about a node. |
| `kubectl describe pod [pod-name]` | Show detailed information and events for a pod. |
| `kubectl logs [pod-name]` | Print the logs for a pod. |
| `kubectl logs -f [pod-name]` | Stream the logs for a pod in real-time. |
| `kubectl exec -it [pod-name] -- /bin/bash` | Get an interactive shell inside a running pod. |
| `kubectl apply -f [filename.yaml]`| Create or update resources from a YAML file. |
| `kubectl delete -f [filename.yaml]`| Delete resources defined in a YAML file. |
| `kubectl delete pod [pod-name]` | Delete a specific pod. |
| `kubectl get services` | List all services in the current namespace. |
| `kubectl get deployments`| List all deployments in the current namespace. |
| `kubectl get configmaps`| List all ConfigMaps. |
| `kubectl get secrets` | List all Secrets. |
| `kubectl get pv` | List all Persistent Volumes. |
| `kubectl get pvc` | List all Persistent Volume Claims. |
| `kubectl get ingress` | List all Ingress resources. |
| `kubectl top nodes` | Show CPU and Memory usage for nodes (requires metrics-server). |
| `kubectl top pods` | Show CPU and Memory usage for pods (requires metrics-server). |

---

## 4. Kubernetes Component YAML Examples

You can save these examples as `.yaml` files and apply them with `kubectl apply -f <filename>.yaml`.

### Deployment

A Deployment manages a set of replica Pods. This example runs 3 replicas of Nginx.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21.0
        ports:
        - containerPort: 80
```

### Service (NodePort)

A Service exposes an application running on a set of Pods. A `NodePort` service exposes the application on a static port on each node's IP.

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
    - protocol: TCP
      port: 80 # Port inside the cluster
      targetPort: 80 # Port on the pod
      nodePort: 30080 # Port on the host machine (node)
```

### ConfigMap

A ConfigMap stores non-confidential data in key-value pairs. Pods can consume them as environment variables or files.

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-app-config
data:
  DATABASE_HOST: "mysql.example.com"
  API_ENDPOINT: "https://api.example.com/v1"
```

### Secret

A Secret stores sensitive data, like passwords or API keys. Data must be base64-encoded.

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-app-secret
type: Opaque
data:
  # Values are base64 encoded: echo -n 'my-super-secret-password' | base64
  DB_PASSWORD: "bXktc3VwZXItc2VjcmV0LXBhc3N3b3Jk"
  API_KEY: "MWYyZDFlMmU2N2Rm"
```

### PersistentVolume (PV)

A PersistentVolume is a piece of storage in the cluster. This example uses `hostPath`, which is suitable for a single-node setup for development/testing.

```yaml
# persistent-volume.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce # Can be mounted by a single node
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/mnt/data/my-pv" # Make sure this directory exists on your node
```

### PersistentVolumeClaim (PVC)

A PersistentVolumeClaim is a request for storage by a user. It consumes a PV.

```yaml
# persistent-volume-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

### Ingress

An Ingress manages external access to services, typically HTTP. It can provide load balancing, SSL termination, and name-based virtual hosting. **Requires an Ingress Controller to be installed in the cluster (e.g., NGINX Ingress Controller).**

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: "myapp.example.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```
