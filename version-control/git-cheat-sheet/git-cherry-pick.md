# git cherry-pick — Apply Specific Commits

```bash
git cherry-pick <commit>              # apply one commit onto current branch
git cherry-pick <c1> <c2> <c3>        # several commits
git cherry-pick <start>^..<end>       # inclusive range
git cherry-pick <start>..<end>        # range excluding <start>
git cherry-pick -n <commit>           # apply without committing (--no-commit)
git cherry-pick -e <commit>           # edit message
git cherry-pick -x <commit>           # append "(cherry picked from commit ...)"
git cherry-pick -s <commit>           # add Signed-off-by
git cherry-pick -m 1 <merge-commit>   # pick a merge commit relative to parent 1
git cherry-pick --ff <commit>         # fast-forward if possible
git cherry-pick -X theirs <commit>    # conflict strategy option
```

## Conflicts
```bash
git add <file>
git cherry-pick --continue
git cherry-pick --skip
git cherry-pick --abort
git cherry-pick --quit
```

## Find what still needs picking
```bash
git cherry -v main feature            # '+' = not in main, '-' = equivalent exists
git log --cherry-pick --right-only --oneline main...feature
```
