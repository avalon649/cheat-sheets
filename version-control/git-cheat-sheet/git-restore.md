# git restore — Restore Working Tree / Index Files

```bash
git restore <file>                       # discard unstaged changes
git restore .                            # discard all unstaged changes
git restore --staged <file>              # unstage (keep working changes)
git restore --staged .                   # unstage everything
git restore --staged --worktree <file>   # unstage AND discard (match HEAD)
git restore -S -W <file>                 # same as above, short flags
git restore --source=<commit> <file>     # restore file from commit
git restore --source=HEAD~2 -- src/
git restore --source=main --worktree <file>
git restore -p <file>                    # interactive, per hunk
git restore --ours <file>                # conflict: take our version
git restore --theirs <file>              # conflict: take their version
git restore -m <file>                    # recreate conflict markers
```
