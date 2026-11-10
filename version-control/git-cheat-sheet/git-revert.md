# git revert — Undo Commits Safely (new commit)

> Use on shared/pushed branches — history is not rewritten.

```bash
git revert <commit>               # new commit that undoes <commit>
git revert HEAD                   # undo last commit
git revert HEAD~3..HEAD           # undo last 3 commits (one revert each)
git revert -n <c1> <c2>           # revert without committing (--no-commit)
git revert --no-edit <commit>     # use default message
git revert -m 1 <merge-commit>    # revert a merge, keep parent 1 as mainline
git revert -s <commit>            # add Signed-off-by
```

## Revert several commits as one
```bash
git revert -n HEAD~3..HEAD
git commit -m "Revert last 3 commits"
```

## Conflicts
```bash
git add <file>
git revert --continue
git revert --skip
git revert --abort
```

## Revert the revert (re-apply)
```bash
git revert <revert-commit>
```
