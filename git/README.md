# 💻 Git

> Frequently used Git commands for version control.

[← Back to Home](../README.md)

---

## 📋 Table of Contents

- [Setup & Configuration](#setup--configuration)
- [Creating & Cloning Repos](#creating--cloning-repos)
- [Basic Workflow](#basic-workflow)
- [Branching](#branching)
- [Merging & Rebasing](#merging--rebasing)
- [Remote Repositories](#remote-repositories)
- [Undoing Changes](#undoing-changes)
- [Stashing](#stashing)
- [Tags & Releases](#tags--releases)
- [Inspection & Comparison](#inspection--comparison)
- [Advanced & Useful Tricks](#advanced--useful-tricks)
- [Git Aliases](#git-aliases)

---

## Setup & Configuration

```bash
# Set user info
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Set default editor
git config --global core.editor vim
git config --global core.editor "code --wait"   # VS Code

# Set default branch name
git config --global init.defaultBranch main

# Set line ending handling
git config --global core.autocrlf input    # Linux/Mac
git config --global core.autocrlf true     # Windows

# View config
git config --list
git config --global --list
git config user.email

# Credential helper
git config --global credential.helper store
git config --global credential.helper cache  # 15 min cache

# SSH key for GitHub
ssh-keygen -t ed25519 -C "you@example.com"
cat ~/.ssh/id_ed25519.pub     # copy to GitHub/GitLab
ssh -T git@github.com         # test connection
```

---

## Creating & Cloning Repos

```bash
# Initialize new repo
git init
git init my-project

# Clone a repo
git clone https://github.com/user/repo.git
git clone https://github.com/user/repo.git my-folder
git clone --depth 1 https://github.com/user/repo.git   # shallow clone
git clone --branch develop https://github.com/user/repo.git
git clone git@github.com:user/repo.git                 # SSH

# Initialize and push to remote
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/user/repo.git
git push -u origin main
```

---

## Basic Workflow

```bash
# Check status
git status
git status -s               # short format

# Stage files
git add file.txt
git add .                   # stage all changes
git add -p                  # interactive staging (patch)
git add -u                  # stage modified/deleted (not new)

# Commit
git commit -m "feat: add new feature"
git commit -am "fix: update config"   # add + commit tracked files
git commit --amend -m "new message"   # amend last commit

# Conventional commit types:
# feat:     new feature
# fix:      bug fix
# docs:     documentation
# style:    formatting
# refactor: code restructuring
# test:     adding tests
# chore:    maintenance
# ci:       CI/CD changes

# View commit log
git log
git log --oneline
git log --oneline --graph --all
git log --oneline -10           # last 10 commits
git log --author="Name"
git log --since="2024-01-01"
git log --grep="feat"
git log -- file.txt             # commits affecting file
```

---

## Branching

```bash
# List branches
git branch             # local
git branch -r          # remote
git branch -a          # all

# Create branch
git branch feature/login
git checkout -b feature/login   # create and switch
git switch -c feature/login     # modern syntax

# Switch branch
git checkout main
git switch main

# Rename branch
git branch -m old-name new-name
git branch -m new-name            # rename current branch

# Delete branch
git branch -d feature/login       # safe delete
git branch -D feature/login       # force delete

# Delete remote branch
git push origin --delete feature/login

# Track remote branch
git checkout -b feature/login origin/feature/login
git switch --track origin/feature/login

# List merged/unmerged branches
git branch --merged
git branch --no-merged
```

---

## Merging & Rebasing

```bash
# Merge a branch into current
git merge feature/login
git merge --no-ff feature/login    # force merge commit
git merge --squash feature/login   # squash all commits into one

# Abort a merge in conflict
git merge --abort

# Rebase current branch onto main
git rebase main
git rebase -i HEAD~3               # interactive rebase (last 3 commits)

# Abort rebase
git rebase --abort

# Continue after resolving conflicts
git add .
git rebase --continue

# Cherry-pick a commit
git cherry-pick abc1234
git cherry-pick abc1234..def5678   # range of commits

# Squash last N commits
git reset --soft HEAD~3
git commit -m "Squashed 3 commits"
```

---

## Remote Repositories

```bash
# View remotes
git remote -v

# Add remote
git remote add origin https://github.com/user/repo.git
git remote add upstream https://github.com/original/repo.git

# Rename / remove remote
git remote rename origin new-name
git remote remove upstream

# Fetch (download without merging)
git fetch origin
git fetch --all
git fetch --prune              # remove deleted remote branches

# Pull (fetch + merge)
git pull
git pull origin main
git pull --rebase              # rebase instead of merge

# Push
git push origin main
git push -u origin main        # set upstream tracking
git push --all                 # push all branches
git push origin feature/login
git push --force-with-lease    # safer force push
git push origin --tags         # push all tags
```

---

## Undoing Changes

```bash
# Discard working directory changes
git checkout -- file.txt      # restore file (legacy)
git restore file.txt          # restore file (modern)
git restore .                 # restore all

# Unstage changes
git reset HEAD file.txt       # legacy
git restore --staged file.txt # modern

# Undo last commit (keep changes staged)
git reset --soft HEAD~1

# Undo last commit (keep changes unstaged)
git reset --mixed HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Revert a commit (creates new commit)
git revert abc1234
git revert HEAD               # revert last commit

# Clean untracked files
git clean -n                  # dry run
git clean -f                  # force remove untracked files
git clean -fd                 # also remove untracked directories
git clean -fX                 # remove only ignored files

# Reflog — recover lost commits
git reflog
git checkout abc1234          # recover specific commit
git reset --hard HEAD@{3}     # reset to reflog position
```

---

## Stashing

```bash
# Save work in progress
git stash
git stash push -m "work in progress: login feature"
git stash push -u              # include untracked files

# List stashes
git stash list

# Apply stash
git stash pop                  # apply and remove latest
git stash apply                # apply but keep in list
git stash apply stash@{2}      # apply specific stash

# Drop stash
git stash drop
git stash drop stash@{1}
git stash clear                # remove all stashes

# Show stash content
git stash show
git stash show -p              # full diff

# Create branch from stash
git stash branch feature/stashed-work
```

---

## Tags & Releases

```bash
# List tags
git tag
git tag -l "v1.*"

# Create tag
git tag v1.0.0                           # lightweight
git tag -a v1.0.0 -m "Version 1.0.0"    # annotated
git tag -a v1.0.0 abc1234 -m "Tagging past commit"

# Push tags
git push origin v1.0.0
git push origin --tags                   # push all tags

# Delete tag
git tag -d v1.0.0
git push origin --delete v1.0.0          # delete remote tag

# Checkout a tag
git checkout v1.0.0
git checkout -b hotfix/1.0.1 v1.0.0     # branch from tag
```

---

## Inspection & Comparison

```bash
# Show changes
git diff                        # working vs staged
git diff --staged               # staged vs last commit
git diff HEAD                   # working vs last commit
git diff main..feature/login    # between branches
git diff v1.0.0..v1.1.0        # between tags

# Show a commit
git show abc1234
git show HEAD
git show HEAD:path/to/file.txt  # file at commit

# Blame
git blame file.txt              # who changed each line
git blame -L 10,20 file.txt     # specific lines

# Search
git grep "TODO"
git grep "TODO" -- '*.py'

# Log file history
git log --follow -p file.txt    # full history with renames
git log --oneline -- file.txt

# Who contributed most
git shortlog -sn                # by number of commits
git shortlog -sn --all
```

---

## Advanced & Useful Tricks

```bash
# Bisect — find commit that introduced a bug
git bisect start
git bisect bad                   # current commit is bad
git bisect good v1.0.0           # last known good
# git bisect good/bad after each test
git bisect reset                 # end bisect

# Submodules
git submodule add https://github.com/user/lib.git lib/
git submodule update --init --recursive
git submodule update --remote    # update to latest

# Sparse checkout (only checkout specific dirs)
git clone --filter=blob:none --sparse https://github.com/user/repo.git
git sparse-checkout add src/ docs/

# Git worktree (multiple working directories)
git worktree add ../hotfix hotfix/1.0.1
git worktree list
git worktree remove ../hotfix

# Bundle (offline transfer)
git bundle create repo.bundle --all
git clone repo.bundle my-repo

# Apply a patch
git format-patch -1 abc1234     # create patch file
git am 0001-my-commit.patch     # apply patch

# Check commit signing
git log --show-signature -1
git verify-commit abc1234
```

---

## Git Aliases

Add to `~/.gitconfig`:

```ini
[alias]
    st = status -s
    co = checkout
    sw = switch
    br = branch
    ci = commit
    lg = log --oneline --graph --all --decorate
    ll = log --oneline -10
    df = diff
    dc = diff --staged
    unstage = restore --staged
    undo = reset --soft HEAD~1
    wip = !git add -A && git commit -m "WIP"
    aliases = !git config --list | grep alias
    contributors = shortlog --summary --numbered
    tags = tag -l
    branches = branch -a
    remotes = remote -v
```

---

[← Back to Home](../README.md)
