# git show — Inspect Objects

```bash
git show                         # last commit (HEAD) with diff
git show <commit>                # specific commit
git show <commit> --stat         # files changed only
git show <commit> --name-only
git show <commit> -- <file>      # commit diff limited to a file
git show <commit>:<file>         # file contents at that commit
git show HEAD~2:src/app.py > old_app.py
git show <tag>                   # tag + tagged commit
git show <branch>                # tip of a branch
git show -s --format=%B <commit> # commit message only
```

## Revision syntax
```text
HEAD          current commit
HEAD~1 / HEAD^      parent
HEAD~3        3 commits back (first-parent)
HEAD^2        second parent of a merge
@{-1}         previously checked-out branch
main@{yesterday}    main as of yesterday (reflog)
HEAD@{2}      reflog entry 2
<branch>@{u}  upstream of branch
:/fix bug     youngest commit whose message matches "fix bug"
```

## Low-level inspection
```bash
git cat-file -t <hash>           # object type
git cat-file -p <hash>           # pretty-print object
git ls-tree HEAD                 # tree of HEAD
git ls-tree -r --name-only HEAD  # all tracked files at HEAD
git ls-files                     # files in the index
git rev-parse HEAD               # full hash of HEAD
git rev-parse --short HEAD
git rev-parse --abbrev-ref HEAD  # current branch name
git rev-parse --show-toplevel    # repo root path
git describe --tags              # nearest tag description
```
