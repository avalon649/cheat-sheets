# git rm / git mv — Remove & Move Files

## Remove
```bash
git rm <file>                    # delete file and stage the deletion
git rm -r <dir>                  # remove directory recursively
git rm --cached <file>           # stop tracking, keep file on disk
git rm -r --cached .             # untrack everything (e.g. after .gitignore update)
git rm -f <file>                 # force remove modified/staged file
git rm -n -r <dir>               # dry run
git rm '*.log'                   # glob (quote it)
```

## Re-apply .gitignore to already-tracked files
```bash
git rm -r --cached .
git add .
git commit -m "Apply .gitignore"
```

## Move / rename
```bash
git mv <old> <new>               # rename and stage
git mv <file> <dir>/             # move into directory
git mv -f <old> <new>            # overwrite destination
git mv -n <old> <new>            # dry run
git mv readme.md README.md       # case-only rename (works on case-insensitive FS)
```
