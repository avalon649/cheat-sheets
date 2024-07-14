# Kubectl Cheat-Sheet
## Config and Cluster Management
COMMAND | DESCRIPTION
---|---
`kubectl cluster-info` | Display endpoint information about the master and services in the cluster
`kubectl config view` |Get the configuration of the cluster
## Resource Management
COMMAND | DESCRIPTION
---|---
`kubectl get all --all-namespaces` | List all resources in the entire Cluster


### List of kubectl Short Names
Short Name | Long Name
---|---
`csr`|`certificatesigningrequests`
`cs`|`componentstatuses`
`cm`|`configmaps`
`ds`|`daemonsets`
`deploy`|`deployments`
`ep`|`endpoints`
`ev`|`events`
`hpa`|`horizontalpodautoscalers`
`ing`|`ingresses`
`limits`|`limitranges`
`ns`|`namespaces`
`no`|`nodes`
`pvc`|`persistentvolumeclaims`
`pv`|`persistentvolumes`
`po`|`pods`
`pdb`|`poddisruptionbudgets`
`psp`|`podsecuritypolicies`
`rs`|`replicasets`
`rc`|`replicationcontrollers`
`quota`|`resourcequotas`
`sa`|`serviceaccounts`
`svc`|`services`
## Logs and Troubleshooting
### Logs

### Executing Commands on Pods

### Networking
`kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash` 
### MySQL 
`kubectl run -it --rm --image=mysql:5.7 --restart=Never mysql-client -- mysql -u USERNAME -h HOSTNAME -p`
### Kubeconfig
`export KUBECONFIG=file1:file2`

## zsh auto-completion

### The kubectl completion script for Zsh can be generated with the command kubectl completion zsh. Sourcing the completion script in your shell enables kubectl autocompletion.

### To do so in all your shell sessions, add the following to your ~/.zshrc file:
```bash
source <(kubectl completion zsh)
```
### Relabel Role Name
```
kubectl label --overwrite nodes <your_node> kubernetes.io/role=<your_new_label>
```

### Describe Kubernetes Resource
```bash
kubect explain <resource> --recursive
```
### Run an Interactive Pod
```bash
kubectl --namespace <namespace-name> run <pod-name> --image=alpine --restart=Never --rm --stdin --tty -- sh
``` 

### Kustomize

```bash
kustomize build nginx | kubectl apply -f -
```

### Decode a secret

```bash
kubectl get secret secret-name -o jsonpath='{.data.value}' | base64 --decode
```

### Label namespace 

```bash
kubectl label namespace default istio-injection=enabled
```

### Show namespace label

```bash
kubectl get namespace default --show-label
```

### Deploy manifest from terminal

```bash
cat <<EOF | kubectl apply -f -
```

### To create the secret, use:
```bash
kubectl create secret generic ssh-keys --from-file=id_rsa=/path/to/.ssh/id_rsa --from-file=id_rsa.pub=/path/to/.ssh/id_rsa.pub
```