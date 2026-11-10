# git remote — Manage Remotes

```bash
git remote                            # list remote names
git remote -v                         # with URLs
git remote show origin                # detailed info (branches, tracking)
git remote add <name> <url>           # add remote
git remote add upstream <url>         # typical for forks
git remote rename <old> <new>
git remote remove <name>              # (alias: rm)
git remote get-url origin
git remote set-url origin <new-url>   # change URL (e.g. HTTPS -> SSH)
git remote set-url --add --push origin <url2>   # push to multiple URLs
git remote prune origin               # delete stale remote-tracking branches
git remote prune origin --dry-run
git remote update                     # fetch all remotes
git remote set-head origin --auto     # set origin/HEAD from remote default branch
```

## Fork workflow
```bash
git clone git@github.com:me/project.git
cd project
git remote add upstream git@github.com:owner/project.git
git fetch upstream
git switch main
git rebase upstream/main              # or: git merge upstream/main
git push origin main
```

## Switch HTTPS → SSH
```bash
git remote set-url origin git@github.com:user/repo.git
```
