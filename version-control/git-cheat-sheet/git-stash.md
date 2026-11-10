# git stash — Shelve Changes Temporarily

```bash
git stash                         # stash tracked changes (staged + unstaged)
git stash push -m "message"       # stash with description
git stash -u                      # include untracked files
git stash -a                      # include untracked + ignored
git stash -k                      # keep staged changes in index (--keep-index)
git stash --staged                # stash only staged changes
git stash push <path>             # stash specific files
git stash -p                      # interactively pick hunks
```

## List / inspect
```bash
git stash list
git stash show                    # summary of latest stash
git stash show -p                 # full diff of latest stash
git stash show -p stash@{2}
git stash show --include-untracked stash@{0}
```

## Apply / remove
```bash
git stash pop                     # apply latest and drop it
git stash pop stash@{2}
git stash apply                   # apply latest, keep it in list
git stash apply --index           # also restore staged state
git stash drop stash@{1}          # delete one stash
git stash clear                   # delete all stashes
git stash branch <new> stash@{0}  # new branch from stash's base, apply it
```

## Restore a single file from a stash
```bash
git checkout stash@{0} -- <file>
git restore --source=stash@{0} -- <file>
```

## Recover a dropped stash
```bash
git fsck --unreachable | grep commit | cut -d' ' -f3 | xargs git log --merges --no-walk --oneline
git stash apply <hash>
```
