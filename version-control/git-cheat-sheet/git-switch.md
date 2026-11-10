# git switch — Change Branches

```bash
git switch <branch>              # switch to existing branch
git switch -                     # switch to previous branch
git switch -c <new>              # create and switch
git switch -c <new> <start>      # create from commit/branch/tag
git switch -C <name>             # create or reset and switch
git switch -c <name> --track origin/<name>
git switch <name>                # auto-creates tracking branch if origin/<name> exists
git switch --detach <commit>     # detached HEAD at commit
git switch --orphan <name>       # new branch with no history
git switch -f <branch>           # discard local changes and switch
git switch -m <branch>           # carry local changes with a 3-way merge
```
