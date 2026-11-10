# git rebase — Reapply Commits on a New Base

> Don't rebase commits that others have already pulled (shared branches).

```bash
git rebase main                  # replay current branch on top of main
git rebase origin/main
git rebase main feature          # check out feature, then rebase onto main
git rebase --onto main old-base feature   # move commits after old-base onto main
git rebase -i HEAD~5             # interactive: edit last 5 commits
git rebase -i main               # interactive: all commits since main
git rebase -i --root             # interactive from the very first commit
git rebase -i --autosquash main  # auto-arrange fixup!/squash! commits
git rebase --autostash main      # stash/unstash dirty tree automatically
git rebase --update-refs main    # also move stacked branches
git rebase -x "make test" main   # run a command after each commit
git rebase --exec "npm test" HEAD~3
git rebase --rebase-merges main  # preserve merge commits
git rebase --committer-date-is-author-date main
```

## During a rebase
```bash
git status                       # see where you are
git add <file>                   # after resolving a conflict
git rebase --continue
git rebase --skip                # drop the current commit
git rebase --abort               # back to pre-rebase state
git rebase --edit-todo           # change the remaining plan
```

## Interactive todo commands
```text
pick   (p)  use commit
reword (r)  use commit, edit message
edit   (e)  use commit, stop to amend
squash (s)  meld into previous commit, combine messages
fixup  (f)  meld into previous, discard this message (-C keeps this one)
drop   (d)  remove commit
exec   (x)  run shell command
break  (b)  stop here (continue with --continue)
label / reset / merge   used with --rebase-merges
```

## Fixup workflow
```bash
git commit --fixup=<commit>
git rebase -i --autosquash <commit>~1
```

## Split a commit
```bash
git rebase -i <commit>~1         # mark commit as 'edit'
git reset HEAD~1                 # unstage its changes
git add -p && git commit -m "part 1"
git add -p && git commit -m "part 2"
git rebase --continue
```

## Undo a finished rebase
```bash
git reflog                       # find pre-rebase HEAD (e.g. HEAD@{5})
git reset --hard HEAD@{5}
git reset --hard ORIG_HEAD       # if nothing else moved HEAD since
```
