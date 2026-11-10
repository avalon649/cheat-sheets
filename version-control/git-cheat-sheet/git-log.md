# git log — View History

```bash
git log                              # full history of current branch
git log --oneline                    # one line per commit
git log --oneline --graph --decorate --all
git log -n 10                        # last 10 commits (also: -10)
git log --stat                       # files changed per commit
git log -p                           # full patch per commit
git log -p -- <file>                 # patches touching a file
git log --follow -- <file>           # history across renames
git log <branch>                     # history of another branch
git log main..feature                # commits in feature not in main
git log feature..main                # commits in main not in feature
git log main...feature --left-right  # commits unique to either side
git log --first-parent               # only mainline (merge) commits
git log --merges                     # only merge commits
git log --no-merges                  # exclude merges
git log --reverse                    # oldest first
```

## Filtering
```bash
git log --author="Name"
git log --committer="Name"
git log --since="2 weeks ago" --until="yesterday"
git log --after=2026-01-01 --before=2026-06-30
git log --grep="fix"                 # search commit messages
git log --grep="fix" -i              # case-insensitive
git log -S"functionName"             # pickaxe: commits adding/removing string
git log -G"regex"                    # commits whose diff matches regex
git log -L 10,20:<file>              # history of line range
git log -L :funcName:<file>          # history of a function
git log -- <path>                    # commits touching a path
git log --diff-filter=D --summary    # commits that deleted files
```

## Formatting
```bash
git log --pretty=format:"%h %an %ar %s"
git log --pretty=format:"%C(yellow)%h%Creset %C(blue)%ad%Creset %s %C(green)(%an)%Creset" --date=short
git log --format="%H"                # full hashes only
git log --date=iso
git log --abbrev-commit
git log --show-signature             # verify signatures
```

## Format placeholders
```text
%H  full hash        %h  short hash       %s  subject
%an author name      %ae author email     %ad author date
%ar author date, relative                 %cn committer name
%cd committer date   %d  ref names        %b  body
```

## Related
```bash
git shortlog -sn                     # commit count per author
git shortlog -sne --no-merges
git rev-list --count HEAD            # number of commits
git whatchanged                      # (legacy) log with file changes
```
