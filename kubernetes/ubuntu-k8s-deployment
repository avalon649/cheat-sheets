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