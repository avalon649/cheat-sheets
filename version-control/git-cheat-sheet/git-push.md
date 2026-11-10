# git push — Upload Commits

```bash
git push                              # push current branch to its upstream
git push <remote> <branch>
git push -u origin <branch>           # push and set upstream (--set-upstream)
git push origin HEAD                  # push current branch to same name
git push origin local:remote          # push local branch to differently named remote
git push --all                        # all local branches
git push --tags                       # all tags
git push --follow-tags                # commits + annotated tags reachable from them
git push origin <tag>                 # single tag
git push origin --delete <branch>     # delete remote branch
git push origin :<branch>             # same (old syntax)
git push origin --delete <tag>        # delete remote tag
git push --dry-run
git push -v
git push --no-verify                  # skip pre-push hook
git push --mirror <url>               # push all refs (mirror)
git push -o ci.skip                   # push option (GitLab skip CI)
```

## Force push (rewritten history)
```bash
git push --force-with-lease           # safe: fails if remote moved unexpectedly
git push --force-with-lease --force-if-includes
git push --force                      # unsafe: overwrites remote blindly
git push origin +<branch>             # force one branch only
```

## Recommended defaults
```bash
git config --global push.autoSetupRemote true
git config --global push.default current
git config --global push.followTags true
```
