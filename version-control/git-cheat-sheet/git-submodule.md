# git submodule — Nested Repositories

```bash
git submodule add <url> <path>               # add submodule
git submodule add -b main <url> <path>       # track a branch
git submodule init                           # register from .gitmodules
git submodule update                         # check out recorded commits
git submodule update --init --recursive      # init + update all (nested too)
git submodule update --remote                # pull latest from tracked branch
git submodule update --remote --merge
git submodule status
git submodule summary
git submodule foreach 'git pull'             # run command in each
git submodule foreach --recursive 'git status -s'
git submodule sync --recursive               # sync URLs after .gitmodules change
git submodule set-url <path> <new-url>
git submodule set-branch -b main <path>
```

## Clone with submodules
```bash
git clone --recurse-submodules <url>
# or after a normal clone
git submodule update --init --recursive
```

## Always recurse
```bash
git config --global submodule.recurse true
git config --global status.submoduleSummary true
```

## Remove a submodule
```bash
git submodule deinit -f <path>
git rm -f <path>
rm -rf .git/modules/<path>
git commit -m "Remove submodule <path>"
```
