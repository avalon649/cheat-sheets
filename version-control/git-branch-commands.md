# Git Branch Commands

## Create a New Branch

### Create a branch (without switching to it)
```bash
git branch <branch-name>
```

### Create a branch from a specific commit
```bash
git branch <branch-name> <commit-hash>
```

### Create a branch from another branch
```bash
git branch <new-branch> <existing-branch>
```

## Switch to a Branch

### Switch to an existing branch
```bash
git checkout <branch-name>
```

### Switch to a branch (newer syntax)
```bash
git switch <branch-name>
```

### Create and switch to a new branch in one command
```bash
git checkout -b <branch-name>
```

### Create and switch to a new branch (newer syntax)
```bash
git switch -c <branch-name>
```

## Delete a Branch

### Delete a local branch (safe - prevents deletion if unmerged)
```bash
git branch -d <branch-name>
```

### Force delete a local branch (even if unmerged)
```bash
git branch -D <branch-name>
```

### Delete a remote branch
```bash
git push origin --delete <branch-name>
```

### Alternative syntax for deleting remote branch
```bash
git push origin :<branch-name>
```

## Useful Branch Commands

### List all local branches
```bash
git branch
```

### List all remote branches
```bash
git branch -r
```

### List all branches (local and remote)
```bash
git branch -a
```

### Show current branch
```bash
git branch --show-current
```

### Rename current branch
```bash
git branch -m <new-name>
```

### Rename a different branch
```bash
git branch -m <old-name> <new-name>
```

## Examples

```bash
# Create a feature branch
git branch feature/user-authentication

# Create and switch to a feature branch
git checkout -b feature/user-authentication

# Switch to main branch
git checkout main

# Delete merged feature branch
git branch -d feature/user-authentication

# Force delete unmerged branch
git branch -D feature/experimental

# Delete remote branch
git push origin --delete feature/user-authentication
```
