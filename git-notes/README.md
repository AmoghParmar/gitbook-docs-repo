# Git Notes

---

## Merge Conflict

A **merge conflict** occurs when we try to merge another branch into our current branch, and **both branches have different changes on the same line of code**. Since Git cannot decide which change to keep, it asks us to resolve the conflict manually.

---

## git add .

```bash
git add .
```

This command is used to stage **all changes**. It moves all files from the **Unstaged Changes** area to the **Staged Changes** area.

---

## git add \<path/fileName\>

```bash
git add <path/fileName>
```

This command is used to stage **only a specific file** instead of staging all changes.

Example:

```bash
git add src/components/Login.vue
```

---

## git branch

```bash
git branch
```

This command is used to view all the local branches and also shows **which branch you are currently working on**.

The current branch is marked with `*`.

Example:

```text
* feature/login
  main
```

---

## git init

```bash
git init
```

This command is used to convert an **empty folder into a Git repository**.

After running this command, Git creates a hidden `.git` folder that stores all the repository history and configuration.

---

## git status

```bash
git status
```

This command is used to check the current status of your repository.

It tells you:

- Which branch you are currently on.
- Which files are **Staged** (shown in green).
- Which files are **Unstaged** (shown in red).
- Which files are untracked.

---

## git push

```bash
git push
```

This command is used to push **only the committed changes** to the remote repository (GitHub).

If a change has not been committed, it will not be pushed.

---

## git pull

```bash
git pull
```

This command is used to perform:

```text
FETCH + MERGE
```

It fetches the latest changes from the remote repository and merges them into your current branch.

---

## git checkout -b feature

```bash
git checkout -b feature
```

This command creates a **new branch** and automatically switches to it.

The newly created branch contains all the code from the branch from which it was created (usually `main`).

---

## git commit -m "\<commit message\>"

```bash
git commit -m "Add login validation"
```

This command creates a commit for the staged changes.

Once the changes are committed, they are ready to be pushed to the remote repository.

---

## git push -u origin feature

```bash
git push -u origin feature
```

This command pushes the **feature branch** to GitHub for the first time and sets it as the upstream branch.

After this, you can simply use:

```bash
git push
```

for future pushes.

---

## git stash

```bash
git stash
```

This command is used to temporarily save your **unfinished work**.

### Real-life Example

Suppose you are working on an issue, and before completing it, your mentor or supervisor asks you to work on another urgent task.

Since your current work is incomplete, you don't want to commit it.

Instead, you use:

```bash
git stash
```

This saves all your current **uncommitted changes** locally so that you can switch branches and work on the urgent task.

After completing the urgent work, you can come back and restore your previous changes.

> `git stash` saves only those changes that have **not been committed** (both staged and unstaged changes).

---

## git stash pop

```bash
git stash pop
```

This command is used to restore the changes that were previously saved using `git stash`.

It takes the latest stashed changes and applies them back to your working directory, allowing you to continue your previous work.

---

## Understanding Merge Conflict (Real-Life Example)

A **merge conflict** happens when **two developers modify the same line of code** in different branches.

### Example

**Person A** and **Person B** both pull the latest code.

Both start working on the same file.

Both modify **the same line**.

### Scenario

```text
PERSON A
    │
Makes changes
    │
Commits
    │
Pushes Successfully
```

```text
PERSON B
    │
Makes changes on the SAME line
    │
Commits
    │
Tries to Push
    │
❌ Push Rejected
```

GitHub rejects Person B's push because the remote repository already contains newer commits from Person A.

Now Person B needs to run:

```bash
git pull
```

During the pull operation, Git tries to merge both changes.

Since **both developers modified the same line differently**, Git cannot merge them automatically.

At this point, a **Merge Conflict** occurs.

```text
PERSON A
    │
Push
    │
────────────► GitHub
                     ▲
                     │
PERSON B
    │
Push ❌ Rejected
    │
git pull
    │
Merge Conflict
    │
Resolve Conflict
    │
Commit
    │
Push Again ✅
```

> **Note:** A merge conflict does **not** happen during `git push`. It usually appears while running `git pull` (which performs Fetch + Merge) or during a manual `git merge`.

---

## git fetch upstream

```bash
git fetch upstream
```

This command is used to **fetch the latest commits and changes** from the **upstream repository**.

It only downloads the latest changes into your local repository.

It **does not merge** those changes into your current branch.

### Flow

```text
Upstream Repository
        │
        ▼
git fetch upstream
        │
        ▼
Latest commits are downloaded
(No merge happens)
```

---

## git remote -v

```bash
git remote -v
```

This command is used to check all the configured **remote repositories**.

It helps you verify whether:

- `origin` is configured.
- `upstream` is configured.
- The URLs of both repositories.

Example:

```text
origin      https://github.com/amogh/project.git (fetch)
origin      https://github.com/amogh/project.git (push)

upstream    https://github.com/company/project.git (fetch)
upstream    https://github.com/company/project.git (push)
```

This command is commonly used to verify whether an **upstream remote already exists**.

---

## What does `-v` mean?

`-v` stands for **Verbose**.

Verbose means **show more detailed information**.

Example:

```bash
git remote
```

Output:

```text
origin
upstream
```

Whereas,

```bash
git remote -v
```

Output:

```text
origin      https://github.com/amogh/project.git (fetch)
origin      https://github.com/amogh/project.git (push)

upstream    https://github.com/company/project.git (fetch)
upstream    https://github.com/company/project.git (push)
```

The `-v` option displays additional details, such as the fetch and push URLs for each remote repository.

---
