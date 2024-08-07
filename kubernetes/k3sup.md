### K3s installation with k3sup

```bash
k3sup install \
--ip 10.0.25.5 \
--tls-san 10.0.25.250 \
--cluster \
--k3s-channel latest \
--k3s-extra-args "--disable servicelb --disable traefik" \
--local-path $HOME/.kube/config \
--user serveradmin \
--merge