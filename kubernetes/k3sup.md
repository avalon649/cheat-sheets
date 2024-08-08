### K3s installation with k3sup

```bash
curl -sLS https://get.k3sup.dev | sh
sudo install k3sup /usr/local/bin/

k3sup --help
```
### Create Kubernetes Cluster

```bash
k3sup install \
--ip 10.0.25.5 \
--tls-san 10.0.25.250 \
--cluster \
--k3s-channel latest \
--k3s-extra-args "--disable servicelb traefik" \
--local-path $HOME/.kube/config \
--user serveradmin \
--merge
```

### Join a Second Server

```bash
k3sup join \
--server \
--host $SERVER_2 \
--user $SERVER_2_USER \
--server-host $SERVER_1 \
--server-user $SERVER_1_USER \
--k3s-channel stable 
```