## Create a new repository on the command line

```bash
echo "# testing" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/avalon649/testing.git
git push -u origin main
```

## Push an existing repository from the command line

```bash
git remote add origin https://github.com/avalon649/testing.git
git branch -M main
git push -u origin main
```