# git worktree — Multiple Working Directories

> Check out several branches at once from one repository.

```bash
git worktree list
git worktree add ../hotfix hotfix-branch      # existing branch
git worktree add -b feature ../feature main   # new branch from main
git worktree add --detach ../review <commit>
git worktree add ../pr origin/pr-branch
git worktree remove ../hotfix
git worktree remove --force ../hotfix         # even with local changes
git worktree prune                            # clean up deleted worktree dirs
git worktree move ../old ../new
git worktree lock ../usb-wt --reason "on removable drive"
git worktree unlock ../usb-wt
git worktree repair
```

## Typical flow
```bash
git worktree add -b fix/urgent ../repo-fix main
cd ../repo-fix
# ... fix, commit, push ...
cd -
git worktree remove ../repo-fix
```
