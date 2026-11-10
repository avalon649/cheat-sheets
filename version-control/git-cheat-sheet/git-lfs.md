# git lfs — Large File Storage

```bash
git lfs install                       # set up hooks (once per user)
git lfs track "*.psd"                 # track pattern (writes .gitattributes)
git lfs track "assets/videos/**"
git lfs untrack "*.psd"
git lfs track                         # list tracked patterns
git add .gitattributes
git lfs ls-files                      # LFS files in current commit
git lfs status
git lfs pull                          # download LFS content for checkout
git lfs fetch --all
git lfs push --all origin
git lfs prune                         # delete old local LFS objects
git lfs env
git lfs migrate import --include="*.zip" --everything   # move history into LFS
git lfs migrate info
```

## Clone without downloading LFS blobs
```bash
GIT_LFS_SKIP_SMUDGE=1 git clone <url>
git lfs pull --include="needed/dir/*"
```
