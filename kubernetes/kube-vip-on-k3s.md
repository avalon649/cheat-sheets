### Kube ViP on K3S

### Create Manifests Folder

```bash
mkdir -p /var/lib/rancher/k3s/server/manifests/

```

### Upload kube-vip RBAC Manifest

```bash
curl https://kube-vip.io/manifests/rbac.yaml > /var/lib/rancher/k3s/server/manifests/kube-vip-rbac.yaml
```
### Generate a kube-vip DaemonSet Manifes

Set the VIP address to be used for the control plane:

```bash
export VIP=192.168.0.40
```

Set the INTERFACE name to the name of the interface on the control plane(s) which will announce the VIP. In many Linux distributions this can be found with the ip a command.

```bash
export INTERFACE=eth0
```

Get the latest version of the kube-vip release by parsing the GitHub API. This step requires that jq and curl are installed.

```bash
KVVERSION=$(curl -sL https://api.github.com/repos/kube-vip/kube-vip/releases | jq -r ".[0].name")
```

### Creating the manifest

For containerd, run the below command:

```bash
alias kube-vip="ctr image pull ghcr.io/kube-vip/kube-vip:$KVVERSION; ctr run --rm --net-host ghcr.io/kube-vip/kube-vip:$KVVERSION vip /kube-vip"
```

### ARP Example for DaemonSet

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

### Example ARP Manifest

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  creationTimestamp: null
  name: kube-vip-ds
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: kube-vip-ds
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: kube-vip-ds
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/master
                operator: Exists
            - matchExpressions:
              - key: node-role.kubernetes.io/control-plane
                operator: Exists
      containers:
      - args:
        - manager
        env:
        - name: vip_arp
          value: "true"
        - name: port
          value: "6443"
        - name: vip_interface
          value: ens160
        - name: vip_cidr
          value: "32"
        - name: cp_enable
          value: "true"
        - name: cp_namespace
          value: kube-system
        - name: vip_ddns
          value: "false"
        - name: svc_enable
          value: "true"
        - name: vip_leaderelection
          value: "true"
        - name: vip_leaseduration
          value: "5"
        - name: vip_renewdeadline
          value: "3"
        - name: vip_retryperiod
          value: "1"
        - name: address
          value: 192.168.0.40
        image: ghcr.io/kube-vip/kube-vip:v0.4.0
        imagePullPolicy: Always
        name: kube-vip
        resources: {}
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
            - NET_RAW
            - SYS_TIME
      hostNetwork: true
      serviceAccountName: kube-vip
      tolerations:
      - effect: NoSchedule
        operator: Exists
      - effect: NoExecute
        operator: Exists
  updateStrategy: {}
status:
  currentNumberScheduled: 0
  desiredNumberScheduled: 0
  numberMisscheduled: 0
  numberReady: 0
```

### Install the kube-vip Cloud Provider

```bash
kubectl apply -f https://raw.githubusercontent.com/kube-vip/kube-vip-cloud-provider/main/manifest/kube-vip-cloud-controller.yaml
```

### Create a global CIDR or IP Range

```bash
kubectl create configmap -n kube-system kubevip --from-literal range-global=192.168.1.220-192.168.1.230
```

### Example Configmap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kubevip
  namespace: kube-system
data:
  cidr-default: 192.168.0.200/29                      # CIDR-based IP range for use in the default Namespace
  range-development: 192.168.0.210-192.168.0.219      # Range-based IP range for use in the development Namespace
  cidr-finance: 192.168.0.220/29,192.168.0.230/29     # Multiple CIDR-based ranges for use in the finance Namespace
  cidr-global: 192.168.0.240/29                       # CIDR-based range which can be used in any Namespace
```