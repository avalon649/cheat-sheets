# git config — Setup & Configuration

## Identity
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

## Scopes
```bash
git config --system <key> <value>   # all users    (/etc/gitconfig)
git config --global <key> <value>   # current user (~/.gitconfig)
git config --local  <key> <value>   # current repo (.git/config) — default
git config --worktree <key> <value> # current worktree
```

## View / edit / remove
```bash
git config --list                    # all effective settings
git config --list --show-origin      # with the file each came from
git config --list --show-scope       # with scope (system/global/local)
git config user.email                # read one key
git config --get-regexp '^alias\.'   # read keys matching a regex
git config --global --edit           # open global config in $EDITOR
git config --global --unset <key>    # remove a key
git config --global --unset-all <key>
```

## Common settings
```bash
git config --global init.defaultBranch main
git config --global core.editor "nvim"
git config --global core.autocrlf input        # Linux/macOS ('true' on Windows)
git config --global core.excludesFile ~/.gitignore_global
git config --global pull.rebase true           # rebase instead of merge on pull
git config --global pull.ff only               # refuse non-fast-forward pulls
git config --global push.default current       # push current branch to same name
git config --global push.autoSetupRemote true  # auto -u on first push
git config --global fetch.prune true           # prune deleted remote branches on fetch
git config --global rebase.autoStash true
git config --global rebase.autoSquash true
git config --global rerere.enabled true        # reuse recorded conflict resolutions
git config --global merge.conflictStyle zdiff3
git config --global diff.algorithm histogram
git config --global color.ui auto
git config --global help.autocorrect prompt
git config --global credential.helper store    # or 'cache --timeout=3600', 'libsecret'
```

## Commit signing
```bash
# GPG
git config --global user.signingkey <KEYID>
git config --global commit.gpgsign true
git config --global tag.gpgsign true

# SSH
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

## Aliases
```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.sw switch
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'restore --staged'
git config --global alias.last 'log -1 HEAD --stat'
git config --global alias.lg "log --graph --oneline --decorate --all"
git config --global alias.amend 'commit --amend --no-edit'
git config --global alias.undo 'reset --soft HEAD~1'
```

## Conditional includes (per-directory identity)
```ini
# ~/.gitconfig
[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work
```
