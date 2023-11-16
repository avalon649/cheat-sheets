#### Create the RBAC settings

```bash
kubectl apply -f https://kube-vip.io/manifests/rbac.yaml
```

#### Set configuration details

```bash
export VIP=10.0.21.200
export INTERFACE=eth0
export KVVERSION=latest
```

#### Creating the manifest

```bash
alias kube-vip="ctr image pull ghcr.io/kube-vip/kube-vip:$KVVERSION; ctr run --rm --net-host ghcr.io/kube-vip/kube-vip:$KVVERSION vip /kube-vip
```

#### ARP Example for DaemonSet

```bash
kube-vip manifest daemonset \
    --interface $INTERFACE \
    --address $VIP \
    --inCluster \
    --taint \
    --controlplane \
    --services \
    --arp \
    --leaderElection
```

#### Example ARP Manifest

```bash
kubectl apply -f https://raw.githubusercontent.com/avalon649/scripts/master/kube-vip-ds.yaml
```