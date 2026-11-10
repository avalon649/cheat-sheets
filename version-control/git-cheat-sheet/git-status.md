# git status — Inspect the Working Tree

```bash
git status                     # full status
git status -s                  # short format
git status -sb                 # short format + branch/ahead/behind info
git status --porcelain         # stable, script-friendly output
git status --porcelain=v2 --branch
git status -u                  # show all untracked files (incl. inside new dirs)
git status -uno                # hide untracked files
git status --ignored           # also show ignored files
git status <path>              # status for a path only
```

## Short-format codes
```text
XY  file
 M  modified in working tree, not staged
M   modified and staged
MM  staged, then modified again
A   added (staged new file)
D   deleted
R   renamed
C   copied
UU  unmerged (conflict)
??  untracked
!!  ignored
```
