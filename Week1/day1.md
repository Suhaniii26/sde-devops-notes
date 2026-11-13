# Git & GitHub — Week 1 Notes

> Beginner-friendly notes with explanations, commands, examples, and practical workflows.

## Table of Contents

- [What is Git?](#what-is-git)
- [Installing & Configuring Git](#installing--configuring-git)
- [Core Concepts](#core-concepts-the-mental-model)
- [The Core Workflow](#the-core-workflow-most-used-commands)
- [`.gitignore` — Excluding Files](#gitignore--excluding-files)
- [Branching](#branching)
- [Merging](#merging)
- [Remotes & GitHub](#remotes--github)
- [Forking vs Cloning vs Branching](#forking-vs-cloning-vs-branching-commonly-confused)
- [Pull Requests](#pull-requests-prs)
- [Undoing Things & Stash](#undoing-things-very-commonly-asked-in-interviews)
- [Rebase & Tags](#advanced-git-for-real-world-jobs--deeper-interviews)
- [GitHub Actions & Workflows](#github-actions--introduction-to-workflows)
- [Branch Protection Rules](#branch-protection-rules)
- [Complete Pull Request Workflow](#collaboration-a-complete-pull-request-workflow)
- [Common Real-World Scenarios](#common-real-world-scenarios-muscle-memory-youll-actually-use)
- [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
- [Interview Q&A](#interview-qa-frequently-asked)
- [Key Takeaways](#key-takeaways)


---

## What is Git?

**Git** is a **distributed version control system (DVCS)**. It tracks changes to files over time so you can:

- See the full history of every change
- Go back to any previous version
- Work on multiple features in parallel (branches)
- Collaborate with others without overwriting each other's work

**Distributed** means every developer has the *entire* history of the project on their own machine, not just the latest snapshot. This is different from older **centralized** systems like SVN, where only the server holds the full history.

**Why Git exists:** Created by **Linus Torvalds** in 2005 to manage the Linux kernel source code, because existing tools were too slow and not distributed.

> **Interview keyword:** Git vs GitHub — **Git is the tool** (version control system, runs locally). **GitHub is a service** (cloud platform that hosts Git repositories + adds collaboration features like PRs, Issues, Actions). GitLab and Bitbucket are alternatives to GitHub.

---

## Installing & Configuring Git

```bash
# Check if Git is installed
git --version

# Set your identity (required before first commit) — do this once per machine
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# See all config settings
git config --list

# Set default branch name to "main" (modern convention, older default was "master")
git config --global init.defaultBranch main

# Set a default editor (optional)
git config --global core.editor "code --wait"
```

`--global` applies to all repos on your machine. Drop `--global` to set config for *only* the current repo (useful if you use a different email for work vs personal projects).

---

## Core Concepts (the mental model)

Git has **three main areas**:

```text
Working Directory  →  Staging Area (Index)  →  Repository (.git)
   (your files)         (git add)                (git commit)

```

| **Area**                     | **What it is**                                                                      |
| ---------------------------- | ----------------------------------------------------------------------------------- |
| **Working Directory**        | The actual files on your disk that you edit                                         |
| **Staging Area / Index**     | A "waiting room" where you prepare exactly what will go into the next commit        |
| **Repository (.git folder)** | The permanent, saved history — once committed, it's part of your project's timeline |

**Why a staging area?** It lets you commit only *part* of your changes. E.g., you changed 3 files but only want to commit 1 right now — you `add` just that one.

A **commit** is a **snapshot** of your staged files at a point in time, with a unique ID (a SHA-1/SHA-256 hash, e.g. `a1b2c3d`), an author, a timestamp, and a message.

---

## The Core Workflow (most-used commands)

### `git init` — start tracking a project

```bash
mkdir my-project
cd my-project
git init
```

This creates a hidden `.git` folder — that's the entire database of your project's history. Deleting `.git` removes all Git history (the files themselves stay).

### `git status` — see what's going on

```bash
git status
```

Shows: which files are modified, staged, or untracked. **This is the command you'll run constantly** — when in doubt, run `git status`.

### `git add` — stage changes

```bash
git add filename.txt        # stage one file
git add file1.txt file2.txt # stage multiple files
git add .                   # stage everything in current directory
git add -A                  # stage everything in the whole repo (including deletions)
git add -p                  # stage changes interactively, hunk by hunk (great for clean commits)
```

### `git commit` — save a snapshot

```bash
git commit -m "Add login form validation"
git commit -am "Fix typo in header"   # -a auto-stages tracked (already-known) files + commits (skips untracked new files)
```

**Good commit message rules (used in real jobs):**

- Short summary line (≤ 50 chars), imperative mood: "Add", "Fix", "Update" — not "Added" or "Adding"
- Optional blank line + longer description if needed
- Many teams use **Conventional Commits**: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`
  ```
  feat: add JWT authentication middleware
  fix: resolve null pointer in user service
  docs: update README with setup instructions

  ```

### `git log` — view history

```bash
git log                      # full history
git log --oneline            # compact, one line per commit
git log --oneline --graph --all   # visual branch graph — very useful
git log -p                   # show the actual code changes (diff) per commit
git log --author="Priya"     # filter by author
git log --since="2 weeks ago"
```

### `git diff` — see exact changes

```bash
git diff                # unstaged changes (working dir vs staging)
git diff --staged       # staged changes (staging vs last commit)
git diff HEAD            # all changes (working dir vs last commit)
git diff branch1 branch2 # compare two branches
```

### `git show` — inspect a specific commit

```bash
git show a1b2c3d
```

---

## `.gitignore` — excluding files

Not everything should be tracked (e.g., `node_modules/`, `.env`, build output, secrets).

**`.gitignore` file example:**

```
# Dependencies
node_modules/
venv/

# Environment variables / secrets
.env
.env.local

# Build output
dist/
build/
*.log

# OS/editor files
.DS_Store
.vscode/
*.pyc
__pycache__/
```

> **Important:** `.gitignore` only works on files Git isn't *already* tracking. If a file is already committed, add it to `.gitignore` AND run:
>
> ```
> git rm --cached filename
> ```
>
> to stop tracking it going forward (the file stays on disk).

**Interview tip:** Never commit secrets (`.env`, API keys, passwords) — if you accidentally do, changing `.gitignore` later does NOT remove it from history. You'd need `git filter-repo` or BFG Repo-Cleaner, and ideally rotate/invalidate the leaked secret immediately.

---

## Branching

A **branch** is just a movable pointer to a commit. `main` (or `master`) is the default branch. Branching lets you work on a feature without affecting the stable code.

```bash
git branch                  # list branches
git branch feature/login    # create a new branch
git checkout feature/login  # switch to it
git checkout -b feature/login  # create AND switch in one command (shortcut)

# Modern equivalent (Git 2.23+):
git switch feature/login     # switch to existing branch
git switch -c feature/login  # create + switch

git branch -d feature/login  # delete a branch (safe — warns if unmerged)
git branch -D feature/login  # force delete (even if unmerged)
```

**Naming conventions** commonly used in jobs:

- `feature/login-page`
- `bugfix/navbar-overlap`
- `hotfix/payment-crash`
- `release/v1.2.0`

### `HEAD`

`HEAD` is a pointer to your **current** branch/commit. When you `checkout`/`switch`, you're moving `HEAD`.

---

## Merging

Combining changes from one branch into another.

```bash
git checkout main
git merge feature/login
```

**Two types of merges:**

1. **Fast-forward merge** — happens when `main` hasn't moved since you branched off. Git just moves the pointer forward. No new commit created.
2. **3-way merge (true merge commit)** — happens when both branches have diverged (both have new commits). Git creates a new **merge commit** with two parents.

```bash
git merge --no-ff feature/login   # force a merge commit even if fast-forward is possible (keeps feature history visible — many teams prefer this)
```

### Merge Conflicts

Happen when the same lines were changed differently in both branches. Git can't auto-decide, so it pauses and marks the conflict in the file:

```
<<<<<<< HEAD
console.log("Hello from main");
=======
console.log("Hello from feature branch");
>>>>>>> feature/login

```

**To resolve:**

1. Open the file, manually edit it to keep what you want (delete the `<<<<<<<`, `=======`, `>>>>>>>` markers)
2. `git add <file>` (marks it as resolved)
3. `git commit` (completes the merge)

```bash
git merge --abort   # cancel a merge in progress and go back to before you started
```

> **Interview question:** "How do you resolve a merge conflict?" → Explain the markers, manual resolution, `git add`, then `git commit`. Mention tools: VS Code's built-in merge editor, or `git mergetool`.

---

## Remotes & GitHub

A **remote** is a version of your repository hosted elsewhere (e.g., GitHub).

```bash
git remote add origin https://github.com/username/repo.git   # link local repo to GitHub repo
git remote -v                # view remotes
git remote remove origin     # unlink

git push -u origin main      # push local commits to GitHub (-u sets upstream, so future pushes can just be `git push`)
git push                     # after upstream is set
git push origin feature/login  # push a specific branch

git pull                     # fetch + merge changes from remote into current branch
git fetch                    # download changes but DON'T merge (safer — review first)
git fetch && git merge origin/main   # equivalent to git pull, but explicit

git clone https://github.com/username/repo.git   # copy a remote repo to your machine (includes full history)
```

**`git pull` vs `git fetch`:**

- `fetch` = "download the news, but don't act on it yet"
- `pull` = `fetch` + `merge` immediately

> **Interview tip:** Many experienced devs prefer `fetch` then review `git log origin/main` before merging, to avoid surprise conflicts.

### SSH Keys (connect to GitHub without typing a password every time)

```bash
# 1. Generate a key pair
ssh-keygen -t ed25519 -C "you@example.com"
# press Enter to accept default location, optionally set a passphrase

# 2. Start the SSH agent and add your key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 3. Copy the PUBLIC key
cat ~/.ssh/id_ed25519.pub
# copy the output

# 4. Add it on GitHub: Settings → SSH and GPG keys → New SSH key → paste

# 5. Test the connection
ssh -T git@github.com
# should say: "Hi username! You've successfully authenticated..."
```

Then use SSH URLs instead of HTTPS: `git@github.com:username/repo.git` instead of `https://github.com/username/repo.git`.

**Why SSH over HTTPS?** No need to enter username/password (or token) on every push; more secure for automation (CI/CD, servers).

---

## Forking vs Cloning vs Branching (commonly confused)

| **Action** | **What it does**                               | **When to use**                                                                        |
| ---------- | ---------------------------------------------- | -------------------------------------------------------------------------------------- |
| **Clone**  | Copy a repo to your local machine              | You have write access (e.g., your own repo, or team repo)                              |
| **Fork**   | Copy a repo to **your own GitHub account**     | You DON'T have write access (e.g., contributing to someone else's open-source project) |
| **Branch** | A parallel line of work *within* the same repo | Working on a feature without touching `main`                                           |

**Open-source contribution flow:** Fork → Clone your fork → Create a branch → Make changes → Push to your fork → Open a Pull Request to the original repo.

---

## Pull Requests (PRs)

A PR is a GitHub (not Git!) feature — a request to merge your branch into another branch, with a review step.

**Typical flow in a job:**

```bash
git checkout -b feature/add-cart
# ... make changes ...
git add .
git commit -m "feat: add shopping cart component"
git push -u origin feature/add-cart
```

Then on GitHub: **Open a Pull Request** → add description → request reviewers → address review comments (push more commits to the same branch, they auto-appear in the PR) → once approved, **Merge**.

**Merge options on GitHub:**

- **Merge commit** — keeps all individual commits + adds a merge commit
- **Squash and merge** — combines all commits into ONE clean commit (popular for keeping `main` history tidy)
- **Rebase and merge** — replays commits on top of `main`, no merge commit, linear history

> **Interview tip:** Know what "squash and merge" does and why teams use it — it keeps `main`'s history clean (one commit per feature) even if your feature branch had 15 messy "wip" commits.

---

## Undoing Things (very commonly asked in interviews)

| **Command**                          | **What it does**                                            | **Danger level**               |
| ------------------------------------ | ----------------------------------------------------------- | ------------------------------ |
| `git restore <file>`                 | Discard unstaged changes in a file (back to last commit)    | Destructive — changes lost     |
| `git restore --staged <file>`        | Unstage a file (keep the changes, just remove from staging) | Safe                           |
| `git reset --soft HEAD~1`            | Undo last commit, keep changes staged                       | Safe                           |
| `git reset --mixed HEAD~1` (default) | Undo last commit, keep changes unstaged                     | Safe                           |
| `git reset --hard HEAD~1`            | Undo last commit AND delete the changes completely          | **Very destructive**           |
| `git revert <commit>`                | Create a NEW commit that undoes a previous commit           | Safe — doesn't rewrite history |
| `git checkout <commit> -- <file>`    | Restore a file to how it was in a specific old commit       | Safe                           |

```bash
git reset --hard HEAD~1    # danger: deletes last commit's changes forever
git revert a1b2c3d          # safe: adds a new "undo" commit, keeps history intact
```

> **Golden rule / interview answer:** Use `git reset` for commits that are **only local and not pushed yet**. Use `git revert` for commits that are **already pushed/shared with others** — because `reset` rewrites history, which breaks things for teammates who already pulled those commits.

### `git stash` — temporarily shelve changes

```bash
git stash                 # save uncommitted changes, clean working directory
git stash list             # see all stashes
git stash pop              # reapply the most recent stash and remove it from the list
git stash apply             # reapply but KEEP it in the stash list
git stash drop               # delete a stash without applying
git stash push -m "wip: login form"   # stash with a custom message
```

**Use case:** You're mid-work on a feature, and suddenly need to switch branches to fix an urgent bug. Stash your work, switch, fix, switch back, `git stash pop`.

---

## Advanced Git (for real-world jobs & deeper interviews)

### `git rebase` — rewrite history onto a new base

```bash
git checkout feature/login
git rebase main
```

Instead of creating a merge commit, rebase **replays** your branch's commits on top of the latest `main`, producing a clean, linear history.

> **Merge vs Rebase — classic interview question:**
>
> - **Merge**: preserves exact history, creates merge commits, non-destructive, safe for shared branches
> - **Rebase**: rewrites commit history into a clean linear line, makes `log` easier to read, but **NEVER rebase a branch that others have already pulled/are working on** — it rewrites commit hashes and breaks everyone else's copy
> - Common rule: "Rebase local/private branches, merge public/shared branches"

### `git tag` — mark release points

```bash
git tag v1.0.0                              # lightweight tag
git tag -a v1.0.0 -m "First stable release"  # annotated tag (recommended — stores author, date, message)
git push origin v1.0.0       # push a single tag
git push origin --tags       # push all tags
```

---



---

## GitHub Actions — Introduction to Workflows

**GitHub Actions** is a GitHub feature used to automate tasks in a repository. For example, it can run tests whenever someone pushes code or opens a pull request.

A **workflow** is an automated process defined in a YAML file inside `.github/workflows/`.

### Important terms

- **Workflow:** The complete automation process.
- **Event / trigger:** The action that starts a workflow, such as `push` or `pull_request`.
- **Job:** A group of steps that run on a runner.
- **Step:** An individual command or action inside a job.
- **Runner:** The machine that executes the job.
- **Action:** A reusable unit of work.

### Example: run a simple workflow on every push

Create this file in your repository:

`.github/workflows/hello.yml`

```yaml
name: Hello Workflow

on: [push]

jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
      - name: Print a message
        run: echo "Hello from GitHub Actions!"
```

**How it works:**

1. `name` gives the workflow a readable name.
2. `on: [push]` runs it whenever code is pushed.
3. `jobs` contains the jobs to execute.
4. `runs-on` chooses the runner environment.
5. `steps` lists the tasks to perform.
6. `run` executes a shell command.

### Common triggers

- `push` — when commits are pushed.
- `pull_request` — when a pull request is opened or updated.
- `workflow_dispatch` — when a user manually starts the workflow.

**What to practise:** Create the YAML file, push it to GitHub, and open the **Actions** tab to see the workflow run.

---

## Branch Protection Rules

Branch protection rules help prevent accidental or unreviewed changes to important branches such as `main`.

Common protections include:

- Require a pull request before merging.
- Require approval from one or more reviewers.
- Require status checks (such as automated tests) to pass.
- Prevent force pushes.
- Prevent deletion of the protected branch.

### Where to find it

Open your GitHub repository:

**Settings → Branches → Add branch ruleset** (or the branch protection rules option shown in your repository).

Choose the `main` branch and enable the protections required by your project. The exact options available can depend on the repository and account plan.

**Why it matters:** It encourages code review, helps keep `main` stable, and prevents changes from being merged before required checks pass.

---

## Collaboration: A Complete Pull Request Workflow

This is a practical sequence to remember when working with a team:

```bash
# 1. Get the latest changes
git switch main
git pull origin main

# 2. Create a separate branch
git switch -c feature/my-change

# 3. Make changes, then stage and commit
git add .
git commit -m "Add my change"

# 4. Push the branch to GitHub
git push -u origin feature/my-change
```

Then on GitHub:

1. Open a pull request from `feature/my-change` into `main`.
2. Add a clear title and description.
3. Ask a teammate to review it.
4. Make any requested changes and push another commit to the same branch.
5. Wait for required checks and approvals.
6. Merge the pull request using the merge method chosen by the team.

After merging:

```bash
git switch main
git pull origin main
git branch -d feature/my-change
```

**Remember:** A pull request is a request to review and merge changes. It is not the same thing as `git push`.

## Common Real-World Scenarios (muscle memory you'll actually use)

**Scenario: You committed to the wrong branch**

```bash
git log                       # note the commit hash
git reset --soft HEAD~1       # undo commit on wrong branch, keep changes staged
git stash                     # stash the changes
git checkout correct-branch
git stash pop
git commit -m "..."
```

**Scenario: You need to update your feature branch with the latest `main`**

```bash
git checkout feature/login
git fetch origin
git rebase origin/main     # or: git merge origin/main (safer if branch is shared)
```

**Scenario: You want to discard ALL local changes and match the remote exactly**

```bash
git fetch origin
git reset --hard origin/main
```

**Scenario: Accidentally committed a large/secret file**

```bash
git rm --cached secrets.env
echo "secrets.env" >> .gitignore
git commit -m "chore: remove secret file from tracking"
# Note: it still exists in OLD commits — rotate the secret immediately,
# and use `git filter-repo` or BFG Repo-Cleaner to purge it from history if needed
```

---

## Quick Reference Cheat Sheet

```bash
# Setup
git init
git clone <url>
git config --global user.name "Name"
git config --global user.email "email"

# Daily workflow
git status
git add <file>  /  git add .
git commit -m "message"
git push
git pull

# Branching
git branch
git switch -c <branch>
git merge <branch>
git branch -d <branch>

# Inspecting
git log --oneline --graph --all
git diff
git show <commit>
git blame <file>

# Undo
git restore <file>
git reset --soft HEAD~1
git revert <commit>
git stash / git stash pop

# Remote
git remote add origin <url>
git push -u origin main
git fetch
```

---

## Interview Q&A (frequently asked)

**Q: What is the difference between Git and GitHub?** A: Git is a distributed version control *tool* that runs locally and tracks file history. GitHub is a cloud *platform* that hosts Git repositories and adds collaboration features (PRs, Issues, Actions, code review).

**Q: What's the difference between `git merge` and `git rebase`?** A: Merge combines two branches' histories with a new merge commit, preserving exact history. Rebase replays your commits on top of another branch, producing linear history, but rewrites commit hashes — never rebase a shared/public branch.

**Q: What's the difference between `git fetch` and `git pull`?** A: `fetch` downloads remote changes without merging them. `pull` does `fetch` + `merge` automatically.

**Q: What is a detached HEAD state?** A: When `HEAD` points directly to a commit instead of a branch (e.g., after `git checkout <commit-hash>`). Any new commits made here aren't on any branch and can be lost unless you create a branch from that point (`git switch -c new-branch`).

**Q: How do you undo the last commit?** A: Depends: if not pushed, `git reset --soft HEAD~1` (keeps changes) or `--hard` (discards changes). If already pushed/shared, use `git revert <commit>` instead, since it doesn't rewrite shared history.

**Q: What is a merge conflict and how do you resolve it?** A: Occurs when Git can't automatically combine changes because the same lines were edited differently on both branches. Resolve by manually editing the conflict markers in the file, then `git add` and `git commit` (or `git merge --continue`).

**Q: What is `.gitignore` and why is it important?** A: A file listing patterns of files/folders Git should not track (e.g., `node_modules/`, `.env`, build artifacts). Keeps the repo clean and prevents committing secrets or huge generated files.

**Q: Explain the three states/areas in Git.** A: Working directory (your edited files) → Staging area/index (what's prepared for the next commit via `git add`) → Repository (permanent history via `git commit`).

**Q: What is `git stash` used for?** A: Temporarily saving uncommitted changes so you can switch context (e.g., branches) cleanly, then restore them later with `git stash pop`.

**Q: What happens to history when you squash-merge a PR on GitHub?** A: All commits in the feature branch are combined into a single commit on the target branch (e.g., `main`), keeping the main branch history clean and readable.

---

## Key Takeaways

- Git has 3 areas: working directory → staging → repository
- `add` stages, `commit` saves a snapshot, `push`/`pull` sync with remote
- Branches are lightweight pointers; merge or rebase to combine work
- `reset` for local/unpushed mistakes, `revert` for shared/pushed mistakes
- `stash` to pause work, `reflog` to recover "lost" work, `bisect` to hunt bugs
- Feature-branch + Pull Request is the most common real-world workflow
