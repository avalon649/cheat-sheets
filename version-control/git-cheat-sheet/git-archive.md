# git archive / git bundle — Export a Repository

## archive (snapshot of files, no history)
```bash
git archive --format=zip -o release.zip HEAD
git archive --format=tar.gz -o v1.0.0.tar.gz v1.0.0
git archive --prefix=project/ -o project.tar.gz HEAD
git archive HEAD src/ | tar -x -C /tmp/out          # export a subdirectory
git archive --remote=<url> HEAD file.txt | tar -x    # from remote (if allowed)
git archive -l                                       # list formats
```

## bundle (full history in a single file — offline transfer)
```bash
git bundle create repo.bundle --all
git bundle create feature.bundle main..feature
git bundle verify repo.bundle
git bundle list-heads repo.bundle
git clone repo.bundle my-repo
git fetch ../repo.bundle main:from-bundle
```

## format-patch / am (email-style patches)
```bash
git format-patch -1 <commit>             # patch for one commit
git format-patch main..feature           # one file per commit
git format-patch -3 -o patches/
git am patches/*.patch                   # apply, preserving author/message
git am --3way patches/*.patch
git am --continue | --skip | --abort
```
