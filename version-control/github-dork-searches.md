# GitHub Dork Search Examples

GitHub search syntax for finding exposed secrets, misconfigs, and interesting code.

---

## Secrets & Credentials

```
# AWS keys
AKIA language:python
AWS_SECRET_ACCESS_KEY filename:.env

# Generic API keys
api_key password filename:.env
secret_key language:javascript

# Private keys
BEGIN RSA PRIVATE KEY filename:*.pem
BEGIN OPENSSH PRIVATE KEY
```

---

## Config & Environment Files

```
# .env files with credentials
filename:.env DB_PASSWORD
filename:.env SMTP_PASSWORD
filename:config.yml password

# Database connection strings
mongodb+srv password language:javascript
postgresql:// password filename:*.py
```

---

## Exposed Infrastructure

```
# Kubernetes configs
filename:kubeconfig clusters
filename:*.kubeconfig server

# Docker with secrets
filename:docker-compose.yml password
filename:Dockerfile ENV SECRET

# SSH config
filename:ssh_config IdentityFile
```

---

## Specific Technologies

```
# JWT secrets
jwt_secret filename:.env
JWT_SECRET language:javascript

# Slack/Discord webhooks
hooks.slack.com/services
discord.com/api/webhooks filename:*.py

# GitHub tokens
ghp_ filename:*.py
github_token language:javascript
```

---

## Code Patterns

```
# Hard-coded IPs
http://10. language:python
http://192.168. filename:*.js

# Debug/test left in prod
TODO password language:python
FIXME secret language:go

# Commented-out credentials
# password= language:python
```

---

## Search Operator Reference

| Operator      | Example                        | Purpose                        |
|---------------|--------------------------------|--------------------------------|
| `filename:`   | `filename:.env`                | Search specific filenames      |
| `extension:`  | `extension:pem`                | Search by file extension       |
| `language:`   | `language:python`              | Filter by language             |
| `org:`        | `org:microsoft password`       | Search within an org           |
| `repo:`       | `repo:user/repo secret`        | Search specific repo           |
| `path:`       | `path:config secret`           | Search within a path           |
| `NOT`         | `api_key NOT test`             | Exclude terms                  |

---

## Direct URL Format

```
https://github.com/search?q=filename:.env+DB_PASSWORD&type=code
https://github.com/search?q=AKIA+language:python&type=code
https://github.com/search?q=BEGIN+RSA+PRIVATE+KEY&type=code
https://github.com/search?q=discord.com/api/webhooks&type=code
```

---

> Use for authorized security research, bug bounties, or finding your own exposed secrets.
> GitHub enforces rate limits and some searches require a logged-in account.
