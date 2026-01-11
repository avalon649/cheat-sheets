### Traefik Service Registration

docker exec consul-server consul kv put traefik/http/routers/service-name/entrypoints/0 "https"
docker exec consul-server consul kv put traefik/http/routers/service-name/rule "Host(\`consul.example.com\`)"
docker exec consul-server consul kv put traefik/http/routers/service-name/middlewares/0 "default-headers"
docker exec consul-server consul kv put traefik/http/routers/service-name/tls "true"
docker exec consul-server consul kv put traefik/http/routers/service-name/service "service-name"
docker exec consul-server consul kv put traefik/http/services/service-name/loadBalancer/servers/0/url "http://10.0.0.15:1234"
docker exec consul-server consul kv put traefik/http/services/service-name/loadBalancer/passHostHeader "true"

### Traefik Service Removal
docker exec consul-server consul kv delete traefik/http/routers/service-name/entrypoints/0
docker exec consul-server consul kv delete traefik/http/routers/service-name/rule
docker exec consul-server consul kv delete traefik/http/routers/service-name/middlewares/0
docker exec consul-server consul kv delete traefik/http/routers/service-name/tls
docker exec consul-server consul kv delete traefik/http/routers/service-name/service
docker exec consul-server consul kv delete traefik/http/services/service-name/loadBalancer/servers/0/url
docker exec consul-server consul kv delete traefik/http/services/service-name/loadBalancer/passHostHeader
