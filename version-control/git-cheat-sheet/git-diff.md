# git diff — Show Changes

```bash
git diff                         # working tree vs staging area (unstaged changes)
git diff --staged                # staging area vs HEAD (alias: --cached)
git diff HEAD                    # working tree vs HEAD (staged + unstaged)
git diff <commit>                # working tree vs commit
git diff <commit1> <commit2>     # between two commits
git diff <branch1>..<branch2>    # between branch tips
git diff <branch1>...<branch2>   # changes on branch2 since it diverged from branch1
git diff -- <path>               # limit to path
git diff <commit> -- <file>
git diff main:file.txt feature:file.txt   # same file across branches
```

## Output options
```bash
git diff --stat                  # summary of changed files
git diff --shortstat
git diff --numstat
git diff --name-only
git diff --name-status
git diff --word-diff             # word-level diff
git diff --color-words
git diff -w                      # ignore all whitespace
git diff --ignore-blank-lines
git diff -U10                    # 10 lines of context
git diff --diff-filter=M         # only modified (A,C,D,M,R,T,U,X,B)
git diff --check                 # warn on whitespace errors / conflict markers
git diff -M                      # detect renames
git diff --no-index a.txt b.txt  # diff two files outside a repo
```

## Patches
```bash
git diff > changes.patch
git apply changes.patch
git apply --check changes.patch
git apply --3way changes.patch
```

## Diff tools
```bash
git difftool
git difftool --tool=meld
git difftool -d main feature     # directory diff
git range-diff main old-feature new-feature   # compare two versions of a branch
```
