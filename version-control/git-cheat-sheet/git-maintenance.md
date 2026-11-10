# git maintenance / gc / fsck — Repository Housekeeping

```bash
git gc                            # garbage collect + pack objects
git gc --aggressive --prune=now   # deep optimization (slow)
git prune                         # remove unreachable loose objects
git fsck                          # verify object integrity
git fsck --full
git count-objects -vH             # repo size information
git repack -a -d                  # repack into a single pack
git maintenance start             # register repo for background maintenance
git maintenance run               # run maintenance tasks now
git maintenance run --task=gc
git maintenance stop
git maintenance unregister
```

## Find largest objects in history
```bash
git rev-list --objects --all \
  | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
  | awk '$1=="blob"' | sort -k3 -n -r | head -20
```

## Remove a file from all history (git-filter-repo)
```bash
# install: pacman -S git-filter-repo   /   pip install git-filter-repo
git filter-repo --path secrets.env --invert-paths
git filter-repo --strip-blobs-bigger-than 50M
git filter-repo --replace-text expressions.txt
git push --force --all && git push --force --tags
```

## Performance
```bash
git config core.fsmonitor true
git config core.untrackedCache true
git config feature.manyFiles true
git commit-graph write --reachable
```
