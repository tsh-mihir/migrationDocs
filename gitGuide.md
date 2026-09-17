# Git Master Reference Guide

> [!IMPORTANT]
> **Prerequisites:** Some commands in this guide assume that you already have two GitHub accounts (personal and work) set up on your machine via SSH, and that your work account has an SSH Host alias named `github-work` configured in your `~/.ssh/config` file. For these specific dual-account commands to function properly, ensure your SSH keys and accounts are set up first.

This guide covers how to manage personal vs. work GitHub accounts, set up new repositories, create branches, push code, and understand essential Git commands.

---

## 1. Managing Personal vs Work GitHub Accounts

### When does your personal (global) account get used?
Your global default is typically set to your personal account (e.g., `youremail@gmail.com`). This is used automatically for **any repo that does not explicitly override it**. That means:
- Any repo whose remote URL uses plain `github.com` (not a custom alias like `github-work`).
- Any repo where you have not run `git config user.email "..."` locally inside that repo.

In other words: **everything defaults to your personal account** unless you deliberately opt a specific repo into the work account using the steps below.

### How to select the work account for a specific repo
You set this **per-repo**, in two separate places that control two different things:
1. **Remote URL** — controls which account *authenticates* (i.e. which SSH key is used to push/pull).
2. **Local git config** — controls which name/email gets *recorded on your commits* as the author.

---

## 2. Setting Up a New Repository

Here are the exact commands you need to run to get everything set up and pushed to a repository.

### Step 2.1. Initialize Git in the specific folder
Move into your project directory and initialize the repository:
```bash
cd path/to/your/repo
git init
```

### Step 2.2. Check and Select your GitHub Account
If you are using the GitHub CLI (`gh`), you can check which accounts are currently active:
```bash
gh auth status
```
If you need to switch or log into your account, run:
```bash
gh auth login
```

### Step 2.3. Set the local commit identity for this repo
If this is a work repository, ensure your commits are tied to your work email. This does **not** touch your global config.
```bash
git config user.name "Your Name"
git config user.email "your_email@company.com"
```

### Step 2.4. Link the Remote Repository
Point the remote at your GitHub repository. (If using a work alias, replace `github.com` with your alias like `github-work`).
```bash
git remote add origin git@github.com:<org-or-username>/<repo-name>.git
```
*(If the remote already exists and you just need to update it, use `git remote set-url origin git@github.com:<org-or-username>/<repo-name>.git`)*

---

## 3. Branching, Committing, and Pushing

### Creating a Branch
Create a new branch for your feature or bug fix:
```bash
git checkout -b feature/your-feature-name
```
**Explanation of flags:**
- `-b`: Stands for "branch". It tells Git to **create** a new branch and immediately switch (checkout) to it in one step. (Equivalent to running `git branch feature/your-feature-name` followed by `git checkout feature/your-feature-name`).

### Committing and Pushing
Stage your files, commit them, and push the branch upstream to open a Pull Request:
```bash
git add .
git commit -m "feat: added new feature"
git push -u origin feature/your-feature-name
```
**Explanation of flags:**
- `.`: The dot in `git add .` tells Git to stage **all** new, modified, and deleted files in the current directory and subdirectories.
- `-m`: Stands for "message". It allows you to pass the commit message directly inline without opening a text editor.
- `-u`: Stands for "upstream" (or `--set-upstream`). It links your local branch to the remote branch (`origin`). This is crucial for the **first push** of a new branch. After running this once, you can just type `git push` or `git pull` in the future on this branch without specifying the remote and branch name.

---

## 4. Other Must-Know / Good-to-Know Commands

Here are some essential Git commands that you will use frequently:

### `git status`
Shows the state of your working directory and staging area. It lets you see which changes have been staged, which haven't, and which files aren't being tracked by Git.
```bash
git status
```

### `git log`
Displays the commit history for the current branch.
```bash
git log
```
*(Tip: Use `git log --oneline` for a more compact view of the history).*

### `git pull`
Fetches the latest changes from the remote repository and merges them into your current local branch.
```bash
git pull
```

### `git diff`
Shows the exact lines of code added or removed in your working directory since your last commit.
```bash
git diff
```
*(Tip: Use `git diff --staged` to see changes that have already been `git add`ed but not yet committed).*

### `git clone`
Downloads an existing repository from GitHub to your local machine.
```bash
git clone git@github.com:<org-or-username>/<repo-name>.git
```

### `git stash`
Temporarily shelves (or stashes) changes you've made to your working copy so you can work on something else (like switching branches) without having to commit half-done work.
```bash
git stash
```
*(To bring the stashed changes back later, use `git stash pop`).*

---

## 5. Advanced / Everyday Developer Workflows

### 5.1. Undoing Mistakes

**Discard uncommitted changes in a specific file:**
If you messed up a file and just want to reset it back to how it was in the last commit:
```bash
git checkout -- <file_name>
```

**Unstage a file you accidentally added:**
If you ran `git add .` but didn't mean to include a specific file:
```bash
git restore --staged <file_name>
```

**Undo the last commit (keep the changes):**
If you committed too early and want to add more things to it, or change the message:
```bash
git reset --soft HEAD~1
```

**Undo the last commit (destroy the changes):**
*Warning: This permanently deletes your unpushed work.*
```bash
git reset --hard HEAD~1
```

### 5.2. Ignoring Files (`.gitignore`)
You don't want to commit everything to GitHub. Passwords, `.env` files, `node_modules`, and compiled build artifacts should be ignored.

Create a file named `.gitignore` in the root of your repository and list the folder/file names to ignore:
```text
# Example .gitignore
node_modules/
.env
build/
*.log
```
Git will completely ignore these files. If a file is already tracked by Git, adding it to `.gitignore` won't untrack it. You have to remove it from Git's cache first: `git rm -r --cached <file_name>`.

### 5.3. Keeping Branches Up To Date
If `main` has moved forward while you were working on your feature branch, you should pull those changes into your branch to avoid merge conflicts later.

While checked out on your feature branch:
```bash
git fetch origin
git merge origin/main
```
*(Alternatively, you can switch to `main`, run `git pull`, switch back to your branch, and run `git merge main`)*.

### 5.4. Handling Merge Conflicts
Sometimes when you pull or merge, Git can't automatically figure out how to combine the code (e.g., you and someone else edited the exact same line).

1. Git will pause the merge and output a "Merge conflict" warning.
2. Open the affected files in your editor (like VS Code). You will see conflict markers:
```text
<<<<<<< HEAD
Your local changes
=======
Changes coming from the branch you are merging
>>>>>>> origin/main
```
3. Delete the markers (`<<<<`, `====`, `>>>>`) and edit the code to look exactly how it should.
4. Stage the resolved files: `git add <file_name>`
5. Complete the merge: `git commit -m "chore: resolve merge conflicts"`

### 5.5. Cleaning Up Branches
Once your Pull Request is merged, you don't need the branch anymore. 

**Delete a local branch:**
```bash
git branch -d feature/your-feature-name
```
*(If it complains that it isn't fully merged and you want to force delete it, use `-D` instead of `-d`)*.

**Delete a remote branch:**
*(Usually, GitHub deletes this automatically when the PR merges, but if you need to do it manually):*
```bash
git push origin --delete feature/your-feature-name
```

---

## 6. Troubleshooting: Push Errors to `main`

### Error: `src refspec main does not match any`
If you see the error `error: src refspec main does not match any` when trying to push (e.g., `git push origin main`), it usually means your initial commit was made to a branch named `master` (Git's old default), but you are trying to push to `main` (the new standard default).

To resolve this, rename your local branch to `main` before pushing:
```bash
git branch -M main
git push -u origin main
```
**Explanation of flags:**
- `-M`: Forces the renaming of the current branch, even if a branch with the new name already exists.

---

## 7. How to Verify Your Setup

If you ever forget which account a repo uses, you can verify it with these commands inside the repo folder:

1. **Check Remote URL (Authentication):**
```bash
git remote -v
```
This should print the host URL (e.g., `github-work` or `github.com`) in both the fetch and push URLs.

2. **Check Commit Author (Identity):**
```bash
git config user.email
```
This should print your active email for this repo. If you run this in a global/default repo, it will print your personal email.

### Quick Reference

| Question | Answer |
|---|---|
| Which account is used by default? | Personal |
| When is the work account used instead? | Only in repos where you've run both the remote URL change and the local `git config` commands above. |
| Does changing one repo affect others? | No — both the remote URL and `git config` changes are local to that repo folder only. |
