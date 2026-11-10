# git clean — Remove Untracked Files

> Destructive. Always dry-run first with `-n`.

```bash
git clean -n                     # dry run: what would be removed
git clean -f                     # remove untracked files
git clean -fd                    # + untracked directories
git clean -fx                    # + ignored files too
git clean -fX                    # only ignored files (e.g. build artifacts)
git clean -fdx                   # everything not tracked (pristine tree)
git clean -i                     # interactive
git clean -f -- <path>           # limit to path
git clean -fd -e '.env'          # exclude pattern
```

## Fully reset a repo to the last commit
```bash
git reset --hard HEAD
git clean -fdx
```
