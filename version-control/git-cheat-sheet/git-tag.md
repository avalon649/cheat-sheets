# git tag — Mark Releases

## Create
```bash
git tag v1.0.0                        # lightweight tag on HEAD
git tag -a v1.0.0 -m "Release 1.0.0"  # annotated tag (recommended)
git tag -a v1.0.0 <commit> -m "msg"   # tag a past commit
git tag -s v1.0.0 -m "msg"            # signed tag
git tag -f v1.0.0                     # move/replace existing tag
```

## List / inspect
```bash
git tag                               # list tags
git tag -l 'v1.*'                     # filter
git tag -n                            # with annotation line
git tag --sort=-v:refname             # newest version first
git tag --contains <commit>
git tag --points-at HEAD
git show v1.0.0
git describe --tags --abbrev=0        # latest tag reachable from HEAD
git tag -v v1.0.0                     # verify signature
```

## Share
```bash
git push origin v1.0.0                # push one tag
git push origin --tags                # push all tags
git push --follow-tags                # push commits + annotated tags
git fetch --tags
```

## Delete
```bash
git tag -d v1.0.0                     # local
git push origin --delete v1.0.0       # remote
git push origin :refs/tags/v1.0.0     # remote (old syntax)
```

## Check out a tag
```bash
git switch --detach v1.0.0
git switch -c hotfix-1.0.1 v1.0.0     # branch from a tag
```
