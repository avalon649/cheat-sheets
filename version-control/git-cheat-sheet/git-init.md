# git init — Create a Repository

```bash
git init                         # init repo in current directory
git init <directory>             # create directory and init repo in it
git init -b main                 # set initial branch name
git init --bare <repo>.git       # bare repo (no working tree, for servers/remotes)
git init --shared=group          # group-writable repo
git init --template=<dir>        # use a custom template directory
git init --object-format=sha256  # SHA-256 object format
```

## Turn an existing folder into a repo and push it
```bash
cd my-project
git init -b main
git add .
git commit -m "Initial commit"
git remote add origin git@github.com:user/my-project.git
git push -u origin main
```
