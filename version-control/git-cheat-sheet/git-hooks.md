# git hooks — Automate Actions

```bash
ls .git/hooks/                         # sample hooks
git config core.hooksPath .githooks    # use a committed hooks directory
chmod +x .githooks/pre-commit
git commit --no-verify                 # skip pre-commit / commit-msg
git push --no-verify                   # skip pre-push
```

## Common hooks
```text
pre-commit          before commit is created (lint/format/test)
prepare-commit-msg  edit default message
commit-msg          validate message
post-commit         after commit
pre-rebase          before rebase
post-checkout       after checkout/switch
post-merge          after merge/pull
pre-push            before push (run tests)
pre-receive / update / post-receive   server side
```

## Example: pre-commit
```bash
#!/usr/bin/env bash
set -e
if git diff --cached | grep -qE 'AKIA[0-9A-Z]{16}'; then
  echo "AWS key detected — aborting commit." >&2
  exit 1
fi
```

## Example: commit-msg (conventional commits)
```bash
#!/usr/bin/env bash
pattern='^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(.+\))?!?: .{1,72}'
grep -qE "$pattern" "$1" || { echo "Bad commit message format" >&2; exit 1; }
```

## pre-commit framework
```bash
pip install pre-commit
pre-commit install
pre-commit run --all-files
pre-commit autoupdate
```
