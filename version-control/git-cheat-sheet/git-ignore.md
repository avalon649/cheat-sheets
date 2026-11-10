# .gitignore / .gitattributes — Ignore & File Rules

## .gitignore patterns
```gitignore
# comment
*.log              # all .log files anywhere
/build             # only top-level build
build/             # any directory named build
!important.log     # negate (re-include)
doc/*.txt          # txt directly inside doc/
doc/**/*.pdf       # pdf anywhere under doc/
**/temp            # temp at any depth
.env
.env.*
!.env.example
node_modules/
__pycache__/
*.py[cod]
.DS_Store
.idea/
.vscode/
```

## Ignore locations
```bash
.gitignore                       # committed, shared
.git/info/exclude                # local to this clone, not committed
git config --global core.excludesFile ~/.gitignore_global   # all repos
```

## Debug ignores
```bash
git check-ignore -v <file>       # which rule ignores a file
git status --ignored
git ls-files --others --ignored --exclude-standard
```

## Stop tracking a file that's now ignored
```bash
git rm --cached <file>
git commit -m "Stop tracking <file>"
```

## Ignore local changes to a tracked file
```bash
git update-index --skip-worktree <file>      # keep local edits (config files)
git update-index --no-skip-worktree <file>
git update-index --assume-unchanged <file>   # perf hint, not for config
git update-index --no-assume-unchanged <file>
git ls-files -v | grep '^S'                  # list skip-worktree files
```

## .gitattributes
```gitattributes
* text=auto
*.sh   text eol=lf
*.bat  text eol=crlf
*.png  binary
*.psd  filter=lfs diff=lfs merge=lfs -text
package-lock.json  -diff linguist-generated
*.md   diff=markdown
```
