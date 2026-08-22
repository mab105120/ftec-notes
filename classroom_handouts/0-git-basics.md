# Introduction to Version Control with Git

## Introduction to the Command Prompt

The **command prompt** (or terminal) is a text-based tool that allows us to interact with our
computer by typing instructions. Instead of clicking through menus and windows, we write
commands that the system executes directly.

- **Why do developers use it?**
  - It is faster and more flexible than graphical interfaces.
  - Many powerful tools (like Git, Docker, and Python virtual environments) only work in the command line.
    - It creates a common language for development across different operating systems.

- **Common commands you will use often:**
  - `pwd` → shows your current location in the filesystem.
  - `ls` → lists the files and folders in your current directory.
  - `cd foldername` → moves into another folder.
  - `cd ..` → moves up one level.
  - `mkdir name` → creates a new folder.

Think of the command prompt as your **steering wheel for programming** — it gives you full
control of where you’re going and what you’re doing.

## The PATH Environment Variable

When you type a command like python or git, how does your computer know where to find
it?

The answer is the **PATH environment variable**. It’s a list of directories (folders) that your
system searches through whenever you type a command.

- **Analogy:** Imagine you’re in a city asking for pizza. The PATH is your phone’s list of
    saved restaurant addresses. If “Mario’s Pizza” is in the list, you’ll find it. If not, you’ll
    get: _“command not found.”_
- **Why is this important?** If Git isn’t in your PATH, the computer won’t know how to
    run it. You’ll need to install it and make sure its location is added to the PATH.

## Why Do We Need Version Control?

Let’s set the scene:

You and three classmates are working on a group project. You start with one file called
project.docx. After a week of editing, the folder looks like this:

- project_final.docx
- project_final_final.docx
- project_really_final.docx
- project_USE_THIS_ONE.docx

Which is the latest version? Whose edits are included? Did someone accidentally delete
content? This chaos happens every time multiple people work on the same file without
version control.

**Version control systems (VCS)** like Git solve this problem by:

- Keeping one shared history of the project.
- Tracking _who_ changed _what_ and _when_.
- Allowing multiple people to edit at the same time without overwriting each other.
- Making it possible to “go back in time” to any earlier version.

## Introducing Git

**Git** is a distributed version control system. Unlike older systems that stored history on a
central server only, Git keeps a complete copy of the project history on _every developer’s
computer_.

- You can work offline and still have the full project history.
- Every developer is independent but can sync with others.
- Git is the standard in the software industry today.

## Git Workflow: Workspace → Staging Area → Commit

Think of Git as a three-step pipeline:

1. **Workspace** – Your actual project folder where you edit files.
2. **Staging area** – A holding area where you select which changes you want to save.
3. **Commit** – A snapshot of the project, permanently saved in Git’s history.

**Commands to remember:**

- `git add file.txt` → stage changes.
- `git commit -m "Message"` → create a snapshot with a description.

This workflow allows you to carefully prepare commits, like packaging files before sending
them.

## The Git Commit Graph

This is one of the most important ideas to understand in Git.

Each commit in Git is like a **node** in a graph. It records:

- A snapshot of the project at that moment.
- A reference (pointer) to the commit(s) that came before it.

Over time, these commits form a **graph** that looks like a chain of history.

Example: A -- B -- C

- Commit A = project creation.
- Commit B = added a new file.
- Commit C = fixed a bug.

Each commit points back to its parent. Together, they form the “timeline” of your project.

Now, when you create a branch, you are simply creating a **new pointer** in this graph. The
graph doesn’t fork because Git copies files — it forks because a new commit starts
pointing in a different direction.

You can visualize the graph in Git:

`git log --oneline --graph --all`

This command shows a tree-like view of your commits and branches.

## Branching

A **branch** is a label pointing to a commit in the graph. The default branch is usually called
main.

When you create a new branch, Git simply creates a new pointer:

git branch feature-login
git checkout feature-login

Now when you commit, the new commits are attached to the feature-login branch, not
main.

- **Why is this powerful?**  
  Branches allow you to work on features independently. Instead of everyone editing
  the same branch, each developer creates their own branch. When the feature is
  ready, it can be merged back into main.

## Merging

Eventually, you want to combine your branch back into main. This is called a **merge**.

### Fast-Forward Merge

If no one else changed main since you branched, Git just moves the main pointer forward.

```bash
main:           A -- B
feature:               \--C--D

After merge:    A -- B -- C -- D
```

No conflict, no extra commit. Just a fast-forward.

### Merge Commit

If both main and your feature branch have new commits, Git creates a **merge commit**.

```bash
main:           A -- B -- E
feature:                \-- C -- D

After merge:    A -- B -- E
                        \
                          C -- D -- M (merge commit)
```

The merge commit ties both histories together.

### Merge Conflicts

If two branches change the same line of the same file, Git doesn’t know which one to keep.
This is a **merge conflict**. Git will pause and mark the file so you can choose manually.

Example conflict marker in a file:

```python
<<<<<<< HEAD
print("Hello from main")
=======
print("Hello from feature branch")
>>>>>>> feature
```

You must edit the file to resolve the conflict, then stage and commit the fix.

## Remote Git

So far, Git is just local. But collaboration requires a **remote repository**. A remote is a copy
of the repository stored on a server that everyone can sync with.

## GitHub

**GitHub** is the most popular hosting service for Git repositories. It adds collaboration tools
such as:

- Issues (for tracking tasks).
- Pull Requests (for reviewing code before merging).
- Wikis and project boards.

## Syncing with a Remote Repository

Once your local repo is connected to a GitHub repo, you can sync changes:

- `git fetch` → download new changes from remote, but don’t apply them.
- `git pull` → fetch and merge changes into your current branch.
- `git push` → upload your local commits to the remote.

## Best Practices in Git

- **Feature branches:** Always create a branch for new work. Keep main stable.
- **Commit often:** Small, descriptive commits make history easier to follow.
- **Pull requests (PRs):** Before merging into main, open a PR on GitHub so teammates
    can review your code.
- **Sync regularly:** Pull often to keep your local branch up to date.
