# Git Cheat Sheet

**Purpose:** Distributed version control: track changes, collaborate, and manage history.  
**Assumptions:** Git ≥ 2.23 recommended (supports `switch`/`restore`).  
**Last updated:** 2026-04-07

---

## Mental model (3 areas)

- **Working tree**: your files on disk
- **Index / staging area**: what will go into the next commit
- **Repository (HEAD)**: committed history

---

## Quick start (new repo or existing)

```sh
# New repo
git init
git add .
git commit -m "Initial commit"

# Clone existing
git clone <url>
cd <repo>
```

---

## Identity & basic config (one-time)

```sh
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"

git config --global init.defaultBranch main
git config --global core.editor "<editor>"
git config --global -l          # list global config
git config -l                   # list effective config
```

### Credential helpers (HTTPS)

```sh
git config --global credential.helper cache   # cache in memory (default 15 min)
git config --global credential.helper store   # save to ~/.git-credentials (plaintext)
```

### SSH key setup (recommended for long-term use)

```sh
ssh-keygen -t ed25519 -C "your-email@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub       # copy this to your Git host (GitHub, Gitea, etc.)
```

If the remote SSH service runs on a non-standard port:

```
# ~/.ssh/config
Host git.example.com
  HostName git.example.com
  User git
  Port 2222
  IdentityFile ~/.ssh/id_ed25519
```

Test connectivity:

```sh
ssh -T git@git.example.com
```

---

## Day-to-day essentials

```sh
git status                      # what changed?
git diff                        # unstaged changes
git diff --staged               # staged changes
git add <path>                  # stage file/dir
git add -p                      # stage interactively (hunks)
git commit -m "Message"         # commit staged changes
git commit                      # open editor for message
git log --oneline --decorate --graph --all
git show <rev>                  # inspect a commit (or tag)
```

Common revisions:

* `HEAD` (current commit), `HEAD~1` (parent), `<branch>`, `<tag>`, `<sha>`

---

## Branching

```sh
git branch                      # list local branches
git branch -vv                   # show tracking info
git switch <branch>              # switch branches (newer)
git checkout <branch>            # switch branches (older)
git switch -c <new-branch>       # create + switch
git branch -m <new-name>         # rename current branch
git branch -d <branch>           # delete (safe; refuses if unmerged)
git branch -D <branch>           # delete (force)
```

Track a remote branch:

```sh
git fetch
git switch -c <branch> --track origin/<branch>
```

---

## Remotes, fetch/pull/push

```sh
git remote -v                   # list remotes
git remote show origin          # details

git fetch                       # update remote-tracking branches
git pull                        # fetch + integrate (merge or rebase per config)
git pull --rebase               # fetch + rebase (keeps linear history)
git push                        # push current branch
git push -u origin <branch>     # set upstream tracking
```

Useful: update your view of deleted remote branches

```sh
git fetch --prune
```

---

## Merge vs rebase (basic)

Merge (preserves branch structure; creates a merge commit if needed):

```sh
git switch main
git merge <feature-branch>
```

Rebase (replays commits on top of another base; rewrites commit IDs):

```sh
git switch <feature-branch>
git rebase main
```

Rule of thumb: **avoid rebasing commits that are already shared/pushed** unless your team explicitly expects it.

---

## Resolve conflicts (typical flow)

```sh
git status                      # see conflicted files
# edit files to resolve markers
git add <resolved-file>         # mark resolved
git commit                      # complete merge
# or, if rebasing:
git rebase --continue
```

Abort if needed:

```sh
git merge --abort
git rebase --abort
```

---

## Stash (park work-in-progress)

```sh
git stash push -m "WIP"         # stash tracked changes
git stash list
git stash show -p stash@{0}     # inspect
git stash pop                   # apply + drop
git stash apply                 # apply (keep stash)
git stash drop stash@{0}
```

Include untracked files:

```sh
git stash push -u -m "WIP"
```

---

## Undo / fix mistakes (choose carefully)

### Unstage (keep file changes)

```sh
git restore --staged <path>     # newer
git reset <path>                # older
```

### Discard local changes in working tree (irreversible)

```sh
git restore <path>              # newer
git checkout -- <path>          # older
```

### Amend last commit (message or staged content)

```sh
git commit --amend
# if you forgot to add something:
git add <path>
git commit --amend
```

### Move branch pointer (reset)

Use with care; can rewrite history.

```sh
git reset --soft HEAD~1         # undo commit; keep staged changes
git reset --mixed HEAD~1        # undo commit; keep working tree changes (default)
git reset --hard HEAD~1         # undo commit; discard changes (danger)
```

### Undo by adding a new commit (safe for shared history)

```sh
git revert <rev>                # create a new commit that undoes <rev>
```

### “I lost a commit” (often recoverable)

```sh
git reflog                      # history of where HEAD/branches pointed
git switch -c rescue <sha>      # recover by creating a branch at the lost commit
```

---

## Tags (releases)

```sh
git tag                         # list tags
git tag -a v1.0.0 -m "v1.0.0"    # annotated tag
git show v1.0.0
git push --tags                 # push tags
git push origin v1.0.0          # push one tag
```

---

## Clean up / housekeeping

```sh
git clean -n                    # preview removing untracked files
git clean -fd                   # remove untracked files/dirs (danger)
git gc                          # garbage-collect/optimize
```

---

## Search & file history

```sh
git grep "search-term"          # search tracked files for a string
git log -p -- <file>            # show commits + diffs for a specific file
git blame <file>                # who changed each line (and when)
```

---

## .gitignore (common patterns)

```gitignore
# OS / editor noise
.DS_Store
Thumbs.db
*.swp
*~

# Language / tool artifacts
__pycache__/
*.pyc
node_modules/
.venv/
venv/

# Build output
*.o
*.log
*.retry

# Secrets — never commit plaintext credentials
.env
*.pem
```

Tip: check [github.com/github/gitignore](https://github.com/github/gitignore) for language-specific templates.

---

## Help (built-in and always accurate)

```sh
git help
git help <command>              # e.g., git help rebase
git <command> -h                # short help
```

---

## References (optional)

* Git documentation: [https://git-scm.com/docs](https://git-scm.com/docs)
* Pro Git (book):     [https://git-scm.com/book/en/v2](https://git-scm.com/book/en/v2)

