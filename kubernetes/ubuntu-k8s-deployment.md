### How To Install Kubernetes Cluster on Ubuntu 22.04

### Set hostname on Each Node

```bash
sudo hostnamectl set-hostname "k8smaster.example.net"
```

### Disable Swap

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

```bash
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

### Set the following Kernel parameters for Kubernetes, run beneath tee command

```bash
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

### Relaod the above changes, run

```bash
sudo sysctl --system
```

### Install Containerd Runtime

```bash
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```

### Enable docker repository

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

### Now, run following apt command to install containerd

```bash
sudo apt update
sudo apt install -y containerd.io
```

### Configure containerd so that it starts using systemd as cgroup.

```bash
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1

sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

### Restart and enable containerd service

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### Add Apt Repository for Kubernetes

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/kubernetes-xenial.gpg

sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

### Install Kubectl, Kubeadm and Kubelet

```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### Initialize Kubernetes Cluster with Kubeadm

```bash
sudo kubeadm init --control-plane-endpoint=k8smaster.example.net
```

#### After the initialization is complete, you will see a message with instructions on how to join worker nodes to the cluster. Make a note of the kubeadm join command for future reference.

#### So, to start interacting with cluster, run following commands on the master node

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Join Worker Nodes to the Cluster

```bash
sudo kubeadm join k8smaster.example.net:6443 --token vt4ua6.wcma2y8pl4menxh2 \
   --discovery-token-ca-cert-hash sha256:0494aa7fc6ced8f8e7b20137ec0c5d2699dc5f8e616656932ff9173c94962a36
```

### Install Calico Network Plugin

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```

### Add a Storage Class

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```

### Make this storage class (local-path) the default:

```bash
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### Failed Scheduling Fix

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```