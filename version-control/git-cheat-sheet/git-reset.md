# git reset — Move HEAD / Unstage / Undo Commits

| Mode | HEAD | Index (staged) | Working tree |
|------|------|----------------|--------------|
| `--soft` | moved | kept | kept |
| `--mixed` (default) | moved | reset | kept |
| `--hard` | moved | reset | **reset (changes lost)** |
| `--keep` | moved | reset | kept if safe, else abort |

```bash
git reset <file>                 # unstage a file (same as git restore --staged)
git reset                        # unstage everything
git reset --soft HEAD~1          # undo last commit, keep changes staged
git reset HEAD~1                 # undo last commit, keep changes unstaged
git reset --hard HEAD~1          # undo last commit and DISCARD its changes
git reset --hard HEAD            # discard all uncommitted changes
git reset --hard origin/main     # make branch identical to remote
git reset --hard <commit>
git reset --keep <commit>        # safer hard reset (keeps local mods)
git reset --merge                # abort a merge, keep unrelated local changes
git reset -p                     # interactively unstage hunks
git reset --hard ORIG_HEAD       # undo last merge/rebase/reset
```

## Squash last N commits into one
```bash
git reset --soft HEAD~3
git commit -m "Combined commit"
```

## Move commits to a new branch
```bash
git branch feature               # save current position
git reset --hard HEAD~3          # move current branch back
git switch feature
```

## Recover from a bad hard reset
```bash
git reflog
git reset --hard HEAD@{1}
```
