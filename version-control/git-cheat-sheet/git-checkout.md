# git checkout — Switch Branches / Restore Files (legacy all-in-one)

> Modern equivalents: `git switch` for branches, `git restore` for files.

## Branches
```bash
git checkout <branch>                # switch branch
git checkout -                       # previous branch
git checkout -b <new>                # create and switch
git checkout -b <new> <start>        # create from start point
git checkout -B <name>               # create or reset and switch
git checkout -t origin/<branch>      # create tracking branch
git checkout --orphan <name>         # branch with no history
git checkout <commit>                # detached HEAD
git checkout <tag>                   # detached HEAD at tag
```

## Files
```bash
git checkout -- <file>               # discard working-tree changes to file
git checkout .                       # discard all working-tree changes
git checkout <commit> -- <file>      # restore file from a commit
git checkout <branch> -- <path>      # copy path from another branch
git checkout -p -- <file>            # interactively discard hunks
git checkout --ours -- <file>        # during conflict: take our side
git checkout --theirs -- <file>      # during conflict: take their side
```
