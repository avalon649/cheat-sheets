# git sparse-checkout — Check Out Part of a Repo

```bash
git sparse-checkout init --cone           # enable (cone mode = directory-based)
git sparse-checkout set src docs          # only these directories
git sparse-checkout add tests             # add another directory
git sparse-checkout list
git sparse-checkout reapply
git sparse-checkout disable               # back to full checkout
git sparse-checkout set --no-cone '/*.md' '!/drafts/'   # pattern mode
```

## Fast monorepo subset clone
```bash
git clone --filter=blob:none --sparse <url>
cd repo
git sparse-checkout set services/api libs/common
```
