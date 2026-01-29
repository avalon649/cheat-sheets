### Traefik Service Registration For Authelia

```bash
docker exec consul-server consul kv put traefik/http/routers/plex/entryPoints/0 "https"
docker exec consul-server consul kv put traefik/http/routers/plex/rule "Host(\`plex.kore-link.net\`)"
docker exec consul-server consul kv put traefik/http/routers/plex/entryPoints/0 "https"
docker exec consul-server consul kv put traefik/http/routers/plex/service "plex"
docker exec consul-server consul kv put traefik/http/routers/plex/tls "true"
docker exec consul-server consul kv put traefik/http/routers/plex/middlewares/0 "authelia"
docker exec consul-server consul kv put traefik/http/services/plex/loadBalancer/servers/0/url "http://10.0.21.25:32400/"
```