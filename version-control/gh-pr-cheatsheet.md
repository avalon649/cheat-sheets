# GitHub PR Cheat Sheet — Terminal (gh CLI)

## List Pull Requests

### Basic listing

```bash
# List open PRs (default)
gh pr list

# List PRs in a specific repo
gh pr list --repo owner/repo-name --state open
gh pr list --repo avalon649/argocd-prod --state open --json number,title

# List closed PRs
gh pr list --state closed

# List all PRs (open + closed)
gh pr list --state all
```

### Filtering

```bash
# Filter by author
gh pr list --author username

# Filter by label
gh pr list --label bug

# Filter by base branch
gh pr list --base main

# Filter by assignee
gh pr list --assignee username

# Filter by search query
gh pr list --search "docker digest"
```

### Output format

```bash
# Show specific fields as JSON
gh pr list --json number,title,state

# Show more results (default is 30)
gh pr list --limit 100

# Table with number, title, branch, and date
gh pr list --json number,title,headRefName,updatedAt \
  --template '{{range .}}#{{.number}} {{.title}} ({{.headRefName}}) - {{.updatedAt}}{{"\n"}}{{end}}'
```

### View a single PR

```bash
# View PR details in terminal
gh pr view 140

# Open PR in browser
gh pr view 140 --web

# View as JSON
gh pr view 140 --json number,title,body,state,mergeable
```

---

## Merge Pull Requests

### Basic merge

```bash
# Merge with a merge commit (standard)
gh pr merge 140 --merge
gh pr merge 140 --repo avalon649/argocd-prod --merge 

# Squash all commits into one
gh pr merge 140 --squash

# Rebase onto base branch
gh pr merge 140 --rebase
```

### Non-interactive (skip prompts)

```bash
gh pr merge 140 --merge --yes
gh pr merge 140 --squash --yes
gh pr merge 140 --rebase --yes
```

### Merge with custom commit message (squash)

```bash
gh pr merge 140 --squash --subject "feat: update n8n docker digest" --body "Automated digest bump"
```

### Merge against a specific repo

```bash
gh pr merge 140 --repo avalon649/argocd-prod --merge --yes
```

### Merge multiple PRs at once

```bash
for pr in 139 140; do
  gh pr merge $pr --repo avalon649/argocd-prod --merge --yes
done
```

### Auto-merge (merges when checks pass)

```bash
# Enable auto-merge for a PR
gh pr merge 140 --merge --auto

# Disable auto-merge
gh pr merge 140 --disable-auto
```

---

## Quick Reference

| Task | Command |
|------|---------|
| List open PRs | `gh pr list` |
| List PRs in repo | `gh pr list --repo owner/repo` |
| View PR details | `gh pr view <number>` |
| Merge commit | `gh pr merge <number> --merge` |
| Squash merge | `gh pr merge <number> --squash` |
| Rebase merge | `gh pr merge <number> --rebase` |
| Skip prompts | `gh pr merge <number> --merge --yes` |
| Auto-merge on CI pass | `gh pr merge <number> --merge --auto` |
| Merge multiple PRs | `for pr in 139 140; do gh pr merge $pr --merge --yes; done` |
