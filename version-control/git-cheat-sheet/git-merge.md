# git merge — Combine Branches

```bash
git merge <branch>               # merge branch into current
git merge --ff-only <branch>     # only fast-forward, else fail
git merge --no-ff <branch>       # always create a merge commit
git merge --squash <branch>      # squash changes into index (commit yourself)
git merge -m "message" <branch>
git merge --no-commit <branch>   # merge but stop before committing
git merge -X ours <branch>       # on conflict prefer our side (strategy option)
git merge -X theirs <branch>     # on conflict prefer their side
git merge -s ours <branch>       # record merge, keep our tree entirely
git merge --allow-unrelated-histories <branch>
git merge origin/main            # merge fetched upstream
```

## Conflicts
```bash
git status                       # list conflicted files
git diff                         # see conflict hunks
git diff --name-only --diff-filter=U   # only conflicted file names
git mergetool                    # open merge tool
git checkout --ours <file>       # take our version
git checkout --theirs <file>     # take their version
git add <file>                   # mark resolved
git merge --continue             # finish merge (or: git commit)
git merge --abort                # cancel and return to pre-merge state
git merge --quit                 # stop merge, keep working tree as is
```

## Conflict markers
```text
<<<<<<< HEAD
our change
||||||| base
original (with conflictStyle=diff3/zdiff3)
=======
their change
>>>>>>> feature
```

## Inspect before merging
```bash
git log --oneline HEAD..<branch>          # commits that will come in
git diff HEAD...<branch>                   # changes that will come in
git merge-base HEAD <branch>               # common ancestor
git merge-tree --write-tree HEAD <branch>  # test merge without touching tree
```
