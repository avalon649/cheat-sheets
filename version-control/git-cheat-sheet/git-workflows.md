# git workflows — Common Recipes & Undo Guide

## Undo cheat table
| Situation | Command |
|-----------|---------|
| Discard changes in a file | `git restore <file>` |
| Unstage a file | `git restore --staged <file>` |
| Fix last commit message | `git commit --amend -m "new"` |
| Add forgotten file to last commit | `git add <file> && git commit --amend --no-edit` |
| Undo last commit, keep changes | `git reset --soft HEAD~1` |
| Undo last commit, drop changes | `git reset --hard HEAD~1` |
| Undo a pushed commit | `git revert <commit>` |
| Undo a merge/rebase/reset | `git reset --hard ORIG_HEAD` |
| Recover anything | `git reflog` then `git reset --hard HEAD@{n}` |

## Feature branch flow
```bash
git switch main
git pull --rebase
git switch -c feature/x
git add -A && git commit -m "feat: x"
git fetch origin
git rebase origin/main
git push -u origin feature/x
```

## Committed to the wrong branch
```bash
git switch -c correct-branch      # new branch keeps the commit
git switch -
git reset --hard HEAD~1           # remove it from the wrong branch
```

## Sync local main with remote exactly
```bash
git fetch origin
git switch main
git reset --hard origin/main
git clean -fd
```

## Rename the default branch master → main
```bash
git branch -m master main
git push -u origin main
git push origin --delete master
git remote set-head origin -a
```

## Change author on past commits
```bash
git rebase -i <commit> -x "git commit --amend --reset-author --no-edit"
```

## Help
```bash
git help <command>
git <command> -h
git help -a          # all commands
git help -g          # concept guides
git help everyday
```
