# git grep — Search Tracked Content

```bash
git grep "pattern"                    # search working tree (tracked files)
git grep -n "pattern"                 # with line numbers
git grep -i "pattern"                 # case-insensitive
git grep -w "word"                    # whole word
git grep -c "pattern"                 # count per file
git grep -l "pattern"                 # file names only
git grep -L "pattern"                 # files NOT matching
git grep -E "regex|alt"               # extended regex
git grep -P "\d{3}"                   # Perl regex
git grep -e foo --and -e bar          # both on same line
git grep --all-match -e foo -e bar    # files containing both
git grep -p "pattern"                 # show enclosing function
git grep -C 2 "pattern"               # 2 lines context
git grep "pattern" -- '*.py'          # limit by pathspec
git grep "pattern" -- ':!vendor/'     # exclude path
git grep "pattern" <commit|branch>    # search a specific revision
git grep "pattern" $(git rev-list --all)   # search all history (slow)
git grep --untracked "pattern"        # include untracked
git grep --cached "pattern"           # search the index
```
