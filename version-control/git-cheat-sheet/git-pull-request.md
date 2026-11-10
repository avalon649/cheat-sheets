# git pull-request — Pull Request Workflow (git + GitHub CLI)

## Branch → push → open PR
```bash
git switch main && git pull
git switch -c feat/my-change
# ... edit ...
git add -A && git commit -m "feat: my change"
git push -u origin feat/my-change
gh pr create --fill                                  # title/body from commits
gh pr create -t "Title" -b "Body" -B main -H feat/my-change
gh pr create --draft
gh pr create -r reviewer1,reviewer2 -a @me -l bug
gh pr create --web                                   # open in browser
```

## View / check out
```bash
gh pr list
gh pr list --author @me --state open
gh pr status
gh pr view 123
gh pr view 123 --web
gh pr diff 123
gh pr checks 123
gh pr checks 123 --watch
gh pr checkout 123                    # check out PR branch locally
```

## Without gh (plain git)
```bash
git fetch origin pull/123/head:pr-123     # GitHub
git switch pr-123
git fetch origin merge-requests/45/head:mr-45   # GitLab
git request-pull main origin feat/my-change     # generate a PR summary (email workflow)
```

## Update a PR
```bash
git add -A && git commit -m "address review"
git push
# or rewrite history then force-push
git rebase -i origin/main
git push --force-with-lease
gh pr edit 123 --add-reviewer user --add-label ready
gh pr ready 123                           # mark draft as ready
```

## Review
```bash
gh pr review 123 --approve
gh pr review 123 --request-changes -b "Please fix X"
gh pr review 123 --comment -b "LGTM overall"
gh pr comment 123 -b "comment text"
```

## Merge / close
```bash
gh pr merge 123 --squash --delete-branch
gh pr merge 123 --rebase
gh pr merge 123 --merge
gh pr merge 123 --auto --squash           # merge when checks pass
gh pr close 123
gh pr reopen 123
```

## Clean up after merge
```bash
git switch main
git pull
git branch -d feat/my-change
git fetch -p
```
