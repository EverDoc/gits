# Git

## Remove local and remote branch

```bash
git push -d origin <branch>
git branch -d <branch>
```

## List all remote branches

```bash
git fetch --all
git branch -a
```

## Checkout the remote branch

```bash
git checkout -b <branch> origin/<branch>
```

## Track local branch to remote

```bash
# the branch existing in remote
git checkout <branch>
git branch --set-upstream-to=origin/<branch>

# the branch burned in local
git checkout <branch>
git push --set-upstream origin <branch>
```

## Revert one file to specific commit

```bash
git checkout <target commit hash like 'ecdf4'> <file>
```

## Revert to specific commit

```bash
git reset --hard <the commit hash>
git push -f origin <the commit hash>:<remote branch>
```