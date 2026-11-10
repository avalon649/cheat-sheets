# git blame — Who Changed Each Line

```bash
git blame <file>
git blame -L 10,30 <file>             # line range
git blame -L :funcName <file>         # a function
git blame <commit> -- <file>          # as of a commit
git blame -w <file>                   # ignore whitespace changes
git blame -M <file>                   # detect moved lines within file
git blame -C -C -C <file>             # detect lines moved/copied from other files
git blame -e <file>                   # show emails
git blame -s <file>                   # suppress author/date
git blame --date=short <file>
git blame --ignore-rev <commit> <file>
git blame --ignore-revs-file .git-blame-ignore-revs <file>
```

## Ignore formatting commits permanently
```bash
echo "<reformat-commit-hash>" >> .git-blame-ignore-revs
git config blame.ignoreRevsFile .git-blame-ignore-revs
```
