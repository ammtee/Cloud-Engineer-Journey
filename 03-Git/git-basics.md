# Git Basics

## What Git Is

Git is a distributed version control system — it tracks changes to files over time, lets multiple people work on the same codebase without overwriting each other, and keeps a full history of every change. GitHub is a hosting platform *for* Git repositories, not Git itself.

## Core Concepts

| Concept | Description |
|---|---|
| **Repository (repo)** | A project tracked by Git, containing all files and their full history |
| **Commit** | A saved snapshot of changes, with a message describing what changed |
| **Branch** | An independent line of development, allowing parallel work without affecting `main` |
| **Remote** | A version of the repo hosted elsewhere (e.g., on GitHub) |
| **Working Directory** | The actual files on disk you're editing |
| **Staging Area (Index)** | Where changes go before being committed — lets you choose exactly what to include |

## The Git Workflow

```
Working Directory  →  Staging Area  →  Local Repository  →  Remote Repository
      (edit)             (git add)         (git commit)         (git push)
```

## Essential Commands

```bash
git init                          # initialize a new repo
git clone <url>                   # copy an existing remote repo locally
git status                        # see what's changed / staged
git add file.txt                  # stage a specific file
git add .                         # stage all changed files
git commit -m "message"           # save a snapshot with a message
git log                           # view commit history
git log --oneline                 # compact commit history
git diff                          # see unstaged changes
git diff --staged                 # see staged changes not yet committed
```

## Working with Remotes

```bash
git remote -v                     # list configured remotes
git remote add origin <url>       # link a local repo to a remote
git push origin main               # push local commits to the remote
git pull origin main                # fetch + merge remote changes into local
git fetch                          # download remote changes without merging
```

## Undoing Changes

```bash
git restore file.txt               # discard unstaged changes to a file
git restore --staged file.txt      # unstage a file (keep the edits)
git reset --soft HEAD~1            # undo the last commit, keep changes staged
git reset --hard HEAD~1            # undo the last commit, discard changes entirely (careful)
git revert <commit-hash>           # create a new commit that undoes a previous one (safe for shared history)
```

**Key distinction:** `reset` rewrites history (dangerous on shared branches), `revert` adds a new commit that undoes changes (safe — preferred once code is pushed/shared).

## .gitignore

Prevents specific files/folders from ever being tracked — critical for excluding secrets, credentials, and build artifacts.

```
# .gitignore example
.env
*.pem
node_modules/
__pycache__/
*.log
.DS_Store
```

## Best Practices

- Write clear, specific commit messages (what changed and why, not just "fix")
- Commit small, logical changes rather than one giant commit at the end of the day
- Never commit secrets, API keys, or `.pem` files — use `.gitignore` from the start of a project
- Pull before you push to avoid unnecessary merge conflicts
- Use `git status` constantly — it's the fastest way to know what state you're in

## Interview Prep

**Q: What's the difference between `git fetch` and `git pull`?**
`git fetch` downloads changes from the remote but doesn't merge them into your current branch — it just updates your local knowledge of the remote's state. `git pull` does a fetch *and* immediately merges those changes into your current branch in one step.

**Q: What's the difference between `git reset` and `git revert`?**
`git reset` moves the branch pointer backward, effectively rewriting history — safe only on local/unshared commits. `git revert` creates a brand-new commit that undoes the changes of a previous commit, preserving history — the safe choice once commits have been pushed and others may have pulled them.

**Q: Why would you use `.gitignore`?**
To prevent specific files from ever being tracked by Git — most importantly secrets (API keys, `.env` files, private keys) that should never end up in version control, but also generated files like build artifacts or dependency folders that don't need to be tracked and would just bloat the repo.
