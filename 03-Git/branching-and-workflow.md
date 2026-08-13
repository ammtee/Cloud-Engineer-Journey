# Git Branching & GitHub Workflow

## Why Branch?

Branches let you develop features, fix bugs, or experiment without touching the stable `main` branch. Each branch is an independent line of history until it's merged back.

## Branching Commands

```bash
git branch                          # list local branches
git branch feature/login             # create a new branch (doesn't switch to it)
git checkout feature/login           # switch to a branch
git checkout -b feature/login        # create AND switch in one step
git switch feature/login              # modern equivalent of checkout for switching
git switch -c feature/login           # modern equivalent of checkout -b
git branch -d feature/login           # delete a branch (safe — only if merged)
git branch -D feature/login           # force-delete a branch (even if unmerged)
```

## Merging

```bash
git checkout main
git merge feature/login              # merge feature/login into main
```

**Fast-forward merge:** happens when `main` hasn't diverged — Git simply moves the pointer forward, no merge commit created.

**Three-way merge:** happens when both branches have diverged — Git creates a merge commit combining both histories.

## Merge Conflicts

Occur when the same lines of a file were changed differently on two branches. Git marks the conflicting sections:

```
<<<<<<< HEAD
This is the version on main
=======
This is the version on feature/login
>>>>>>> feature/login
```

Resolve by manually editing the file to the correct final state, then:

```bash
git add resolved-file.txt
git commit
```

## Common Branching Strategies

| Strategy | Description |
|---|---|
| **Git Flow** | `main` (production) + `develop` (integration) + feature/release/hotfix branches — structured, common in larger teams |
| **GitHub Flow** | Simpler: `main` is always deployable, feature branches merge directly via Pull Requests |
| **Trunk-Based Development** | Very short-lived branches, frequent merges to `main`, often paired with feature flags |

For most personal and small-team projects, **GitHub Flow** (branch → commit → PR → merge → deploy) is the practical default.

## Pull Requests (GitHub)

A Pull Request (PR) is a request to merge one branch into another, with a review step attached. Typical flow:

```bash
git checkout -b feature/add-readme-badges
# make changes
git add .
git commit -m "Add tech stack badges to README"
git push origin feature/add-readme-badges
# open a Pull Request on GitHub from this branch into main
# after review/approval, merge via GitHub UI
```

PRs enable code review, CI checks (automated tests/linting), and a clear history of *why* a change was made — not just *what* changed.

## Rebasing (vs. Merging)

```bash
git checkout feature/login
git rebase main
```

Rebasing replays your branch's commits on top of the latest `main`, producing a linear history instead of a merge commit. Useful for keeping history clean, but **never rebase commits that have already been pushed and shared** — it rewrites commit hashes and breaks collaborators' history.

## Best Practices

- Never commit directly to `main` on a team project — always branch, then PR
- Keep branches short-lived — long-lived branches accumulate painful merge conflicts
- Use descriptive branch names (`feature/user-auth`, `fix/login-bug`) instead of generic ones
- Write PR descriptions explaining *why*, not just restating the diff
- Delete branches after merging to keep the repo clean

## Interview Prep

**Q: What's the difference between merging and rebasing?**
Merging combines two branches' histories with a new merge commit, preserving exactly what happened and when — non-destructive. Rebasing replays your commits on top of another branch's latest state, producing a cleaner, linear history, but it rewrites commit hashes — which makes it unsafe for commits that have already been pushed and shared with others.

**Q: How do you resolve a merge conflict?**
Git marks the conflicting sections directly in the file with `<<<<<<<`, `=======`, and `>>>>>>>` markers. You manually edit the file to the correct final version, remove the markers, then `git add` the resolved file and complete the commit (or `git rebase --continue` if mid-rebase).

**Q: What's a Pull Request, and why use one instead of pushing directly to `main`?**
A Pull Request proposes merging one branch into another and creates a review checkpoint — teammates can comment, request changes, and CI can run automated tests before the code ever reaches `main`. Pushing directly to `main` skips that safety net entirely, which is risky on any team project.

**Q: What's the difference between `git branch -d` and `git branch -D`?**
`-d` is a safe delete — Git refuses if the branch has unmerged commits, protecting you from losing work. `-D` force-deletes regardless of merge status, which can permanently lose commits that exist only on that branch.
