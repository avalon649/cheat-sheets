# git commit — Record Changes

```bash
git commit                          # open editor for message
git commit -m "message"             # inline message
git commit -m "title" -m "body"     # title + body paragraphs
git commit -a -m "message"          # stage tracked changes and commit
git commit -v                       # show diff in the editor
git commit --amend                  # edit last commit (message and/or content)
git commit --amend --no-edit        # add staged changes to last commit, keep message
git commit --amend -m "new message"
git commit --amend --reset-author --no-edit
git commit --allow-empty -m "trigger CI"
git commit --no-verify -m "msg"     # skip pre-commit / commit-msg hooks
git commit -S -m "msg"              # signed commit (GPG/SSH)
git commit -s -m "msg"              # add Signed-off-by trailer
git commit --author="Name <email>" -m "msg"
git commit --date="2026-01-01T12:00:00" -m "msg"
git commit --fixup=<commit>         # create fixup! commit for autosquash
git commit --squash=<commit>        # create squash! commit for autosquash
git commit --trailer "Co-authored-by: Name <email>" -m "msg"
git commit <file> -m "msg"          # commit only specific file(s)
```

## Conventional commit message format
```text
<type>(<scope>): <short summary>

<body: what and why>

<footer: BREAKING CHANGE: ..., Closes #123>
```

```text
types: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert
```
