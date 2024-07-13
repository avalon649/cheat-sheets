# k3s Installation Guide

Lightweight, easy, fast Kubernetes distribution with a very small footprint

https://k3s.io


### Install Master

```bash
curl -sfL https://get.k3s.io | sh -s - --disable traefik --disable=servicelb --write-kubeconfig-mode 644 --node-name K3S-Server --bind-address 10.0.21.130 --kube-controller-manager-arg bind-address=0.0.0.0 --kube-proxy-arg metrics-bind-address=0.0.0.0 --kube-scheduler-arg bind-address=0.0.0.0 --etcd-expose-metrics true --kubelet-arg containerd=/run/k3s/containerd/containerd.sock
```

```bash
--tls-san lb-ip
```

### Install Worker

Grab token from the master node to be able to add worked nodes to it: 

```bash
cat /var/lib/rancher/k3s/server/node-token
```

Install k3s on the worker node and add it to our cluster:

```bash
curl -sfL https://get.k3s.io | K3S_NODE_NAME=k3s-worker-01 K3S_URL=https://<IP>:6443 K3S_TOKEN=<TOKEN> sh - 
```

### Kubeconfig

Kubernetes configuration file can be found under 

```bash
cat /etc/rancher/k3s/k3s.yml
```

### Uninstall k3s

If you installed K3s with the help of the install.sh script, an uninstall script is generated during installation. The script is created on your node at 

(master)

```bash 
/usr/local/bin/k3s-uninstall.sh
``` 

(node)

```bash
/usr/local/bin/k3s-agent-uninstall.sh
 ```

### Additional information

You can change the settings of k3s by changing the service settings e.g. with 

```bash
nano /etc/systemd/system/k3s.service
```

Make sure to restart the service afterwards: 

```bash
systemctl restart k3s
```

In many cases you can just run the installer with different variables again and it will configure your cluster accordingly without deleting it in the first place.