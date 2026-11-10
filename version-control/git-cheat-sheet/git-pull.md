# git pull — Fetch and Integrate

> `git pull` = `git fetch` + `git merge` (or `git rebase` with `--rebase`).

```bash
git pull                          # fetch + merge upstream of current branch
git pull <remote> <branch>
git pull --rebase                 # fetch + rebase local commits on top
git pull --rebase=interactive
git pull --ff-only                # fail unless fast-forward possible
git pull --no-ff
git pull --squash
git pull --autostash              # stash/unstash dirty working tree
git pull --no-commit
git pull --all                    # fetch all remotes
git pull --tags
git pull --recurse-submodules
git pull upstream main            # sync fork from upstream
git pull --allow-unrelated-histories origin main
```

## Set default pull behavior
```bash
git config --global pull.rebase true     # always rebase
git config --global pull.rebase false    # always merge
git config --global pull.ff only         # only fast-forward
```

## Undo a bad pull
```bash
git reset --hard ORIG_HEAD
# or
git reflog
git reset --hard HEAD@{1}
```
