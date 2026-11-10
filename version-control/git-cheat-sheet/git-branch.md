# git branch — Manage Branches

## List
```bash
git branch                       # local branches
git branch -r                    # remote-tracking branches
git branch -a                    # all branches
git branch -v                    # with last commit
git branch -vv                   # with upstream + ahead/behind
git branch --merged              # merged into current branch
git branch --no-merged           # not yet merged
git branch --contains <commit>   # branches containing a commit
git branch --list 'feat/*'       # filter by pattern
git branch --sort=-committerdate # most recently updated first
git branch --show-current        # current branch name
```

## Create
```bash
git branch <name>                # create from HEAD (does not switch)
git branch <name> <commit|branch|tag>
git branch --track <name> origin/<name>
git switch -c <name>             # create and switch (see git-switch.md)
```

## Rename
```bash
git branch -m <new>              # rename current branch
git branch -m <old> <new>        # rename another branch
git branch -M main               # force rename
```

## Delete
```bash
git branch -d <name>             # delete (only if merged)
git branch -D <name>             # force delete
git push origin --delete <name>  # delete remote branch
git branch -dr origin/<name>     # delete stale remote-tracking ref
```

## Upstream
```bash
git branch -u origin/<name>                  # set upstream for current branch
git branch --set-upstream-to=origin/<name> <local>
git branch --unset-upstream
```

## Cleanup merged branches
```bash
git branch --merged main | grep -vE '^\*|main|master|develop' | xargs -r git branch -d
git fetch -p && git branch -vv | awk '/: gone]/{print $1}' | xargs -r git branch -D
```
