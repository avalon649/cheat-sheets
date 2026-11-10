# git add — Stage Changes

```bash
git add <file>              # stage a file
git add <dir>/              # stage a directory
git add .                   # stage everything under current directory
git add -A                  # stage all changes in repo (new, modified, deleted)
git add -u                  # stage modified + deleted only (no new files)
git add '*.py'              # stage by glob (quote it)
git add -p                  # interactively stage hunks (patch mode)
git add -p <file>
git add -i                  # interactive add menu
git add -N <file>           # intent-to-add (tracked, content not staged)
git add -f <file>           # force-add an ignored file
git add -n .                # dry run — show what would be added
git add --chmod=+x <file>   # stage executable bit
git add --renormalize .     # re-apply line-ending/attributes rules
```

## Patch mode keys (`git add -p`)
```text
y  stage this hunk              n  skip this hunk
a  stage this + rest in file    d  skip this + rest in file
s  split hunk into smaller      e  manually edit hunk
q  quit                         ?  help
```
