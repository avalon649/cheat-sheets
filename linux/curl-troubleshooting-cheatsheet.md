# curl Troubleshooting Cheat Sheet

A practical reference for diagnosing connectivity, latency, TLS, DNS, and API
issues with `curl`. All commands assume `https://myapp.com` as the target —
swap in your own host.

---

## 1. Is it up? Status & reachability

```bash
# Just the HTTP status code (nothing else)
curl -s -o /dev/null -w "%{http_code}\n" https://myapp.com

# Headers only — fast check without downloading the body
curl -I https://myapp.com

# Fail loudly in scripts (non-zero exit on 4xx/5xx)
curl -fsS https://myapp.com/health || echo "DOWN"

# Force a fresh connection, no cache
curl -H "Cache-Control: no-cache" -I https://myapp.com
```

---

## 2. Where is the time going? Latency breakdown

```bash
# Full timing profile — pinpoints DNS vs connect vs TLS vs server
curl -s -o /dev/null -w "\
dns:        %{time_namelookup}s\n\
tcp:        %{time_connect}s\n\
tls:        %{time_appconnect}s\n\
ttfb:       %{time_starttransfer}s\n\
total:      %{time_total}s\n\
size:       %{size_download} bytes\n\
http:       %{http_code}\n" \
https://myapp.com
```

**Reading it:** high `dns` = resolver problem; high `tcp` = network/firewall;
high `tls` = cert/handshake; high `ttfb` with low connect = slow backend.

---

## 3. DNS / routing issues — bypass and pin

```bash
# Test a host by hitting a SPECIFIC IP (bypass DNS entirely)
curl --resolve myapp.com:443:10.0.0.42 -v https://myapp.com/health

# Same idea, simpler, lets you set any Host header
curl -H "Host: myapp.com" http://10.0.0.42/health

# Force IPv4 or IPv6 to isolate dual-stack problems
curl -4 https://myapp.com
curl -6 https://myapp.com

# Which IP did it actually connect to?
curl -s -o /dev/null -w "%{remote_ip}:%{remote_port}\n" https://myapp.com
```

---

## 4. TLS / certificate troubleshooting

```bash
# Full handshake + cert chain details
curl -vI https://myapp.com 2>&1 | grep -E "SSL|subject|issuer|expire|TLS"

# Is the cert the problem? Compare with and without verification
curl -k https://myapp.com        # ignores cert — if this works, it's a cert/trust issue
curl    https://myapp.com        # verifies — fails on bad/expired/self-signed

# Provide a custom CA bundle (internal CA)
curl --cacert /etc/ssl/internal-ca.pem https://internal.myapp.com

# Pin to a TLS version to spot protocol mismatches
curl --tlsv1.2 --tls-max 1.2 https://myapp.com
```

---

## 5. Redirects & "it works in browser but not curl"

```bash
# Show the full redirect chain
curl -sIL https://myapp.com | grep -i -E "HTTP/|location"

# Follow redirects and report how many hops + final URL
curl -s -o /dev/null -L -w "redirects=%{num_redirects} final=%{url_effective}\n" https://myapp.com

# Mimic a browser when a server blocks default curl UA
curl -A "Mozilla/5.0" https://myapp.com
```

---

## 6. Connection hangs / timeouts

```bash
# Never let curl hang — caps connect and total time
curl --connect-timeout 5 --max-time 15 https://slow.myapp.com

# Retry transient failures (flaky network, rolling deploy)
curl --retry 5 --retry-delay 2 --retry-connrefused https://myapp.com/ready

# Test raw TCP port reachability (no HTTP)
curl -v telnet://10.0.0.42:5432   # connects = port open; refused = closed/firewalled
```

---

## 7. Inspecting what the server actually sends

```bash
# Dump response headers to a file while saving body separately
curl -D headers.txt -o body.html https://myapp.com

# See request AND response headers inline
curl -v https://myapp.com 2>&1 | grep -E "^[<>]"

# Check for specific headers (caching, CORS, security)
curl -sI https://myapp.com | grep -i -E "cache-control|etag|access-control|x-"

# Is gzip actually working?
curl -H "Accept-Encoding: gzip" -I https://myapp.com | grep -i content-encoding
```

---

## 8. API debugging (auth, payloads, error bodies)

```bash
# 401/403? Show the body too — APIs often explain the failure in JSON
curl -s -w "\nHTTP %{http_code}\n" -H "Authorization: Bearer $TOKEN" \
  https://api.myapp.com/me

# Verify the exact bytes you're sending
curl -v -X POST https://api.myapp.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"test"}' 2>&1 | grep -A5 "Content-Type"

# Pipe to jq, pretty-print JSON (errors included)
curl -s -H "Authorization: Bearer $TOKEN" https://api.myapp.com/data | jq .
```

---

## 9. Through proxies / load balancers

```bash
# Force through a proxy to test egress rules
curl -x http://proxy.local:3128 -v https://myapp.com

# Which backend served me? (if app echoes a hostname/pod header)
for i in $(seq 1 10); do
  curl -s https://myapp.com/whoami
done | sort | uniq -c     # shows load-balancer distribution
```

---

## 10. Quick reusable health probe (script-ready)

```bash
check() {
  local url=$1
  local code time
  read code time < <(curl -s -o /dev/null -w "%{http_code} %{time_total}" \
    --connect-timeout 5 --max-time 15 "$url")
  echo "$url -> HTTP $code in ${time}s"
}
check https://myapp.com/health
check https://api.myapp.com/ready
```

---

## Most useful `-w` variables for diagnosis

| Variable | Tells you |
|----------|-----------|
| `%{http_code}` | Final status code |
| `%{time_namelookup}` | DNS resolution time |
| `%{time_connect}` | TCP connect time |
| `%{time_appconnect}` | TLS handshake done |
| `%{time_starttransfer}` | Time to first byte (backend speed) |
| `%{time_total}` | End-to-end time |
| `%{remote_ip}` | IP curl actually hit |
| `%{num_redirects}` | Redirect hops |
| `%{size_download}` | Bytes received |
| `%{ssl_verify_result}` | Cert verification result (0 = OK) |
| `%{url_effective}` | Final URL after redirects |

---

## Golden debugging combo

```bash
curl -v -L --connect-timeout 5 --max-time 20 \
  -w "\n--- http=%{http_code} ip=%{remote_ip} dns=%{time_namelookup} ttfb=%{time_starttransfer} total=%{time_total}\n" \
  https://myapp.com
```

`-v` shows the conversation, `-L` follows redirects, the timeouts stop hangs,
and `-w` gives you a one-line summary at the end.

---

## Common flag reference

| Flag | Meaning |
|------|---------|
| `-s` / `-sS` | Silent / silent-but-show-errors |
| `-o` / `-O` | Output to file / use remote filename |
| `-I` / `-i` | Headers only / include headers with body |
| `-v` | Verbose (request, response, TLS) |
| `-L` | Follow redirects |
| `-f` | Fail (non-zero exit) on HTTP errors |
| `-H` | Add a request header |
| `-w` | Custom output format (timings, status) |
| `-k` | Skip TLS verification (dev/lab only) |
| `--resolve` | Pin host:port to a specific IP |
| `--connect-timeout` | Max time to establish connection |
| `--max-time` | Max time for the whole operation |
| `--retry N` | Retry failed requests N times |
| `-x` | Use a proxy |
| `-A` | Set User-Agent |
| `-u` | Basic auth (user:pass) |
