# Git Settings and Essential Commands

This document provides a concise guide to essential Git commands and configurations to streamline your workflow. It includes installation instructions, global configuration settings, and useful tips for working with Git and GitHub.

## Installing Git and Gitk
To get started with Git, install it along with Gitk, a graphical Git repository browser.

```bash
sudo apt install -y git gitk
```

## Configuring Git Globally
Set up your Git environment with global configurations for user information, behavior, and productivity enhancements.

### Set Username and Email
Configure your identity for commits.

```bash
git config --global user.name "FIRST_NAME LAST_NAME"
git config --global user.email "my_email@example.com"
```

### Configure Global default branch name
```bash
git config --global init.defaultBranch main
```

### Enable Automatic Color Output
Enable colorized output for better readability of Git commands.

```bash
git config --global color.ui auto
```

### Configure Line Ending Handling
Ensure consistent line endings across different operating systems (recommended for Linux/macOS).

```bash
git config --global core.autocrlf input
```

### Enable Multiple Hotfixes (Git Flow)
Allow multiple hotfix branches in Git Flow.

```bash
git config --global gitflow.multi-hotfix true
```

### Disable Tagging on Hotfix Finish (Git Flow)
Prevent automatic tagging when finishing a hotfix in Git Flow.

```bash
git config --global gitflow.hotfix.finish.notag true
```

### Set Default Text Editor
Specify your preferred text editor for commit messages (e.g., VS Code, Vim, or Nano).

```bash
git config --global core.editor "code --wait"  # For Visual Studio Code
# or
git config --global core.editor "vim"         # For Vim
# or
git config --global core.editor "nano"        # For Nano
```

### Enable Automatic Rebase for Pulls
Set `git pull` to automatically rebase instead of merge, keeping a cleaner history.

```bash
git config --global pull.rebase true
```

### Configure Useful Aliases
Aliases simplify repetitive Git commands. Add these to your global configuration.

```bash
git config --global alias.st "status"
git config --global alias.co "checkout"
git config --global alias.br "branch"
git config --global alias.cm "commit -m"
git config --global alias.lg "log --graph --oneline --decorate --all"
git config --global alias.unstage "reset HEAD --"
git config --global alias.last "log -1 HEAD"
```

## Common Git Commands
Here are some frequently used Git commands for managing repositories:

### Initialize a Repository
Create a new Git repository in the current directory.

```bash
git init
```

### Clone a Repository
Download a repository from a remote source (e.g., GitHub).

```bash
git clone <repository-url>
```

### Check Repository Status
View the current state of your working directory and staging area.

```bash
git status
```

### Stage Changes
Add files to the staging area for the next commit.

```bash
git add <file>
# or
git add .  # Stage all changes
```

### Commit Changes
Save staged changes to the repository with a message.

```bash
git commit -m "Your commit message"
```

### View Commit History
Display the commit history with a graphical view.

```bash
git log --graph --oneline --decorate
# or
git log --graph --oneline --decorate --all  # Include all branches
```

### Create and Switch Branches
Create a new branch and switch to it.

```bash
git checkout -b <branch-name>
```

### Push Changes
Upload local commits to a remote repository.

```bash
git push origin <branch-name>
```

### Pull Changes
Fetch and merge changes from a remote repository.

```bash
git pull origin <branch-name>
```

### Merge Branches
Combine changes from one branch into another.

```bash
git merge <branch-name>
```

### Stash Changes
Temporarily save uncommitted changes to work on something else.

```bash
git stash save "Your stash message"  # Save changes with a descriptive message
# or
git stash  # Save changes without a message
```

List all stashed changes.

```bash
git stash list
```

Apply the most recent stashed changes and keep them in the stash.

```bash
git stash apply
# or
git stash apply stash@{n}  # Apply a specific stash (n is the stash index from git stash list)
```

Apply the most recent stashed changes and remove them from the stash.

```bash
git stash pop
```

### Cherry-Pick Commits
Apply a specific commit from one branch to another.

```bash
git cherry-pick <commit-hash>  # Apply the specified commit to the current branch
```

### Rebase a Branch
Rebase a branch onto another (e.g., when the base branch has been updated).

```bash
git rebase <base-branch>  # Rebase the current branch onto the specified base branch
```

If conflicts occur during rebase, resolve them and continue:

```bash
git rebase --continue
```

To abort a rebase:

```bash
git rebase --abort
```

## Extra Tip: Configuring SSH for GitHub
To securely authenticate with GitHub, generate an SSH key and add it to the ssh-agent. Follow the detailed instructions provided by GitHub:

[Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
