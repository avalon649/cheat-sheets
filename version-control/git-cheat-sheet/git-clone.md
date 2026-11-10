# git clone — Copy a Repository

```bash
git clone <url>                          # clone into ./<repo-name>
git clone <url> <directory>              # clone into a specific directory
git clone -b <branch> <url>              # clone and check out a specific branch
git clone --single-branch -b <branch> <url>
git clone --depth 1 <url>                # shallow clone (latest commit only)
git clone --shallow-since=2025-01-01 <url>
git clone --filter=blob:none <url>       # partial clone (blobs fetched on demand)
git clone --filter=tree:0 <url>          # treeless clone
git clone --recurse-submodules <url>     # also clone submodules
git clone --bare <url>                   # bare clone
git clone --mirror <url>                 # mirror (all refs; backups/migration)
git clone -o upstream <url>              # name remote 'upstream' instead of 'origin'
git clone --sparse <url>                 # start with sparse-checkout enabled
```

## URL formats
```bash
git clone https://github.com/user/repo.git
git clone git@github.com:user/repo.git
git clone ssh://user@host:2222/path/repo.git
git clone /path/to/local/repo
git clone file:///path/to/local/repo
```

## Deepen a shallow clone
```bash
git fetch --depth=50
git fetch --unshallow
```
