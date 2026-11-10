# git fetch — Download Without Merging

```bash
git fetch                         # fetch from default remote (origin)
git fetch <remote>
git fetch <remote> <branch>
git fetch --all                   # all remotes
git fetch -p                      # prune deleted remote branches (--prune)
git fetch -P                      # prune deleted tags (--prune-tags)
git fetch --tags                  # fetch all tags
git fetch --no-tags
git fetch --depth=1               # shallow fetch
git fetch --unshallow             # convert shallow clone to full
git fetch --dry-run
git fetch origin main:main        # update local main without checking it out (ff only)
git fetch origin pull/123/head:pr-123   # fetch a GitHub PR into branch
```

## After fetching
```bash
git log --oneline HEAD..origin/main   # incoming commits
git diff HEAD...origin/main           # incoming changes
git merge origin/main                 # or: git rebase origin/main
git status -sb                        # ahead/behind counts
```
