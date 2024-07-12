### K3sup command to create a k3s cluster on the master node

bash
k3sup install --ip 10.0.21.60 --tls-san 10.0.21.245 --cluster --user serveradmin --local-path ~/.kube/config --context k3s-ha --k3s-extra-args "disable servicelb --node-ip 10.0.21.60"


### Apply rbac manifest

bash
kubectl apply -f  https://kube-vip.io/manifests/rbac.yaml

### Command for pulling kubevip image

bash
ctr image pull ghcr.io/kube-vip/kube-vip:latest


### Apply kubevip loabalancer

bash
alias kube-vip="ctr run --rm --net-host ghcr.io/kube-vip/kube-vip:latest vip /kube-vip"

kube-vip manifest daemonset \
    –arp \
    –interface eth0 \
    –address 10.0.21.245 \
    –controlplane \
    –leaderElection \
    –taint \
    –inCluster | tee /var/lib/rancher/k3s/server/manifests/kube-vip.yaml 


### K3Sup join additional nodes to the cluster

bash
k3sup join \
    --ip 10.0.21.61 \
    --user serveradmin \
    --sudo \
    --k3s-channel stable \
    --server \
    --server-ip 10.0.21.245 \
    --server-user serveradmin \
    --sudo \
    -- k3s-extra-arg "disable servicelb --node-ip=10.0.21.61"	
    