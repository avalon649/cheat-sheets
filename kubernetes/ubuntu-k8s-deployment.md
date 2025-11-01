### Step 1: Prepare All Nodes

1. Disable Swap (if not already done)

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

2. Update and install dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
```

3. Enable kernel modules & sysctl settings for networking

```bash
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

### Step 2: Install containerd on all nodes

1. Install containerd:

```bash
sudo apt update
sudo apt install -y containerd
```
2. Create containerd config file and restart containerd:

```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### Step 3: Configure containerd for Kubernetes CRI compatibility

We need to make sure containerd uses systemd cgroup driver for Kubernetes compatibility.

`Edit /etc/containerd/config.toml and set:`

```bash
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true
```

You can do this with sed:

```bash
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
```

### Step 4: Install Kubernetes components on all nodes

```bash
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

sudo bash -c 'cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF'

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### Step 5: Disable swap on all nodes (again if needed)

Just double check:

```bash
sudo swapoff -a
```

### Step 6: Initialize Kubernetes Control Plane (on master node)

Use this command on the master node, specifying containerd socket:

```bash
sudo kubeadm init --cri-socket /run/containerd/containerd.sock --pod-network-cidr=10.244.0.0/16
```

### Step 7: Setup kubeconfig on master node

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Step 8: Install a Pod Network Add-on

Flannel example:

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### Step 9: Join worker nodes to the cluster

On master node, get join command:

```bash
kubeadm token create --print-join-command
```

Run the output command on each worker node (also specifying containerd socket if needed):

```bash
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash> --cri-socket /run/containerd/containerd.sock
```

### Step 10: Verify the cluster

On master node:

```bash
kubectl get nodes
```

You should see all 3 nodes in Ready status.
Optional: Allow scheduling on master node (not recommended for production)

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```