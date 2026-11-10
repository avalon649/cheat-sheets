# git reflog — Recover Lost Work

> The reflog records every move of HEAD/branches locally (default ~90 days).

```bash
git reflog                        # HEAD history
git reflog show <branch>          # a branch's history
git reflog --date=iso
git reflog -n 20
git log -g --oneline              # reflog in log format
git reflog expire --expire=now --all   # purge reflog (dangerous)
git reflog delete HEAD@{2}
```

## Recovery recipes
```bash
# Undo a reset / rebase / amend
git reflog
git reset --hard HEAD@{3}

# Restore a deleted branch
git reflog | grep <branch-name>
git branch <branch-name> <hash>

# Recover a commit not in any reflog (dangling)
git fsck --lost-found
git fsck --unreachable --no-reflogs | grep commit
git show <hash>
git branch recovered <hash>
```
