# Tuesday Git Notes

These notes summarize the Git concepts covered during today's DevOps lab. They serve as a quick reference for understanding how Git tracks changes, supports collaboration, and enables safe software development.

---

## Working Directory vs Staging vs History

Understanding these three areas is essential to using Git effectively.

### Working Directory

The **Working Directory** is where you create, edit, and delete files on your local machine. Changes made here are not yet tracked as part of your project's history.

#### Example

```bash
echo "Hello, DevOps!" >> README.md
```

At this point, the change exists only in your working directory.

---

### Staging Area (Index)

The **Staging Area** is where you prepare changes before creating a commit. Only staged files become part of the next commit.

Stage a single file:

```bash
git add README.md
```

Stage every modified file:

```bash
git add .
```

Think of the staging area as a shopping basket—you decide exactly what will be included in your next commit.

---

### Commit History

The **Commit History** is a permanent record of your project's changes.

Create a commit:

```bash
git commit -m "docs: update README"
```

View your project's history:

```bash
git log --oneline
```

Each commit represents a snapshot of the project at a specific point in time.

---

## Git Lifecycle

A typical Git workflow follows this sequence:

```text
Working Directory
        │
        │ git add
        ▼
 Staging Area
        │
        │ git commit
        ▼
 Commit History
        │
        │ git push
        ▼
 GitHub Repository
```

---

## Checking Repository Status

Before performing most Git operations, check the repository status.

```bash
git status
```

`git status` tells you:

- The current branch.
- Modified files.
- Staged files.
- Untracked files.
- Whether your branch is ahead or behind the remote branch.

---

## Branching Rules

Branches allow developers to work independently without affecting the main codebase.

### Good Practices

- Create one branch for each feature or bug fix.
- Use meaningful branch names.
- Keep each branch focused on a single task.
- Test your work before merging.
- Delete branches after they have been merged.

### Branch Naming Examples

```text
feature/login-page
feature/user-profile
bugfix/navbar-overflow
hotfix/payment-timeout
docs/update-readme
```

Create and switch to a branch:

```bash
git switch -c feature/login-page
```

Switch to an existing branch:

```bash
git switch develop
```

---

## Pull Request Expectations

A Pull Request (PR) is a request to merge your work into another branch, typically `develop` or `main`.

### Why Pull Requests Matter

Pull Requests allow teams to:

- Review code before merging.
- Catch bugs early.
- Share knowledge.
- Maintain code quality.
- Encourage collaboration.

### Before Opening a Pull Request

- Ensure your code builds successfully.
- Run all required tests.
- Write a clear title and description.
- Keep the Pull Request focused on one change.
- Request a review.
- Address review comments before merging.

---

## Typical Git Workflow

A common Git workflow looks like this:

```bash
# Check repository status
git status

# Create a feature branch
git switch -c feature/my-feature

# Make your changes

# Stage changes
git add .

# Commit changes
git commit -m "feat: implement my feature"

# Push the branch
git push -u origin feature/my-feature

# Open a Pull Request
```

---

## Common Beginner Mistakes

- Forgetting to run `git status`.
- Forgetting to stage files before committing.
- Committing directly to the `main` branch.
- Using vague commit messages such as "Update" or "Fix".
- Mixing unrelated changes into one commit.
- Forgetting to pull the latest changes before starting new work.

---

## Git Cheat Sheet

| Command | Purpose |
| ---------- | --------- |
| `git status` | View repository status |
| `git add <file>` | Stage a specific file |
| `git add .` | Stage all changes |
| `git commit -m "message"` | Commit staged changes |
| `git log --oneline` | View commit history |
| `git switch branch` | Switch branches |
| `git switch -c branch` | Create and switch to a new branch |
| `git pull` | Download the latest changes |
| `git push` | Upload commits to GitHub |

---

## Key Takeaways

- Git tracks changes through the **Working Directory**, **Staging Area**, and **Commit History**.
- Always check your repository with `git status`.
- Stage only the changes you intend to commit.
- Write meaningful commit messages that describe your work.
- Use feature branches to isolate changes.
- Open Pull Requests to encourage collaboration and code review.
- Keep commits small, focused, and easy to understand.
