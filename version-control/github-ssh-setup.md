# GitHub SSH Authentication Setup (Linux)

## 1. Generate an SSH Key

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Press Enter to accept the default path (`~/.ssh/id_ed25519`).

## 2. Start the SSH Agent and Add Your Key

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

## 3. Copy Your Public Key

```bash
cat ~/.ssh/id_ed25519.pub
```

## 4. Add the Key to GitHub

1. Go to **GitHub > Settings > SSH and GPG keys**
2. Click **New SSH key**
3. Paste the public key and save

## 5. Test the Connection

```bash
ssh -T git@github.com
```

You should see: `Hi <username>! You've successfully authenticated...`

## 6. Clone Repos with SSH

```bash
git clone git@github.com:username/repo.git
```

To switch an existing repo from HTTPS to SSH:

```bash
git remote set-url origin git@github.com:username/repo.git
```
