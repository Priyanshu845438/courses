
# 🚀 Basic Git & Version Control

## Table of Contents
- [What is Git?](#what-is-git)
- [Installation and Setup](#installation-and-setup)
- [Git Fundamentals](#git-fundamentals)
- [Basic Commands](#basic-commands)
- [Working with Repositories](#working-with-repositories)
- [Branching and Merging](#branching-and-merging)
- [Remote Repositories](#remote-repositories)
- [GitHub Basics](#github-basics)
- [Best Practices](#best-practices)
- [Common Workflows](#common-workflows)

---

## 🎯 What is Git?

**Git** is a distributed version control system that tracks changes in files and coordinates work among multiple developers.

### Key Features:
- **Version Control**: Track changes over time
- **Distributed**: Every copy is a full repository
- **Branching**: Create separate development lines
- **Merging**: Combine changes from different branches
- **Collaboration**: Multiple developers can work together

### Git vs Other Version Control Systems

| Feature | Git | SVN | Mercurial |
|---------|-----|-----|-----------|
| **Type** | Distributed | Centralized | Distributed |
| **Speed** | Fast | Slower | Fast |
| **Offline Work** | Yes | Limited | Yes |
| **Branching** | Excellent | Basic | Good |

---

## 🛠 Installation and Setup

### Installing Git

**Windows:**
- Download from [git-scm.com](https://git-scm.com)
- Or use: `winget install Git.Git`

**macOS:**
```bash
# Using Homebrew
brew install git

# Using Xcode Command Line Tools
xcode-select --install
```

**Linux:**
```bash
# Ubuntu/Debian
sudo apt update && sudo apt install git

# CentOS/RHEL
sudo yum install git

# Arch
sudo pacman -S git
```

### Initial Configuration

```bash
# Set your name and email (required)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Set default editor
git config --global core.editor "code --wait"  # VS Code
# git config --global core.editor "nano"       # Nano
# git config --global core.editor "vim"        # Vim

# Check your configuration
git config --list
git config --global --list
```

### Useful Global Settings

```bash
# Better colors
git config --global color.ui auto

# Better diff tool
git config --global diff.tool vimdiff

# Cache credentials (Windows/Mac)
git config --global credential.helper store

# Line ending settings
git config --global core.autocrlf true   # Windows
git config --global core.autocrlf input  # Mac/Linux

# Show original state in merge conflicts
git config --global merge.conflictstyle diff3
```

---

## 🏗 Git Fundamentals

### Git Areas and States

```
Working Directory  →  Staging Area  →  Repository
     (modified)         (staged)        (committed)
         ↓                  ↓               ↓
    git add            git commit      git push
```

### File States

- **Untracked**: New files not yet tracked by Git
- **Modified**: Tracked files that have been changed
- **Staged**: Files ready for the next commit
- **Committed**: Files safely stored in Git database

### Basic Workflow

```bash
# 1. Make changes to files
echo "Hello World" > hello.txt

# 2. Stage changes
git add hello.txt

# 3. Commit changes
git commit -m "Add hello.txt file"

# 4. Push to remote (if applicable)
git push origin main
```

---

## 🔧 Basic Commands

### Repository Initialization

```bash
# Create a new Git repository
git init

# Initialize with specific branch name
git init --initial-branch=main

# Clone an existing repository
git clone https://github.com/username/repository.git

# Clone to specific directory
git clone https://github.com/username/repository.git my-project
```

### Checking Status and History

```bash
# Check repository status
git status

# Short status format
git status -s

# View commit history
git log

# Compact log format
git log --oneline

# Graphical log
git log --graph --oneline --all

# Show specific number of commits
git log -n 5

# Show commits by author
git log --author="John Doe"

# Show commits in date range
git log --since="2023-01-01" --until="2023-12-31"
```

### Adding and Committing

```bash
# Add specific file
git add filename.txt

# Add multiple files
git add file1.txt file2.txt

# Add all files in current directory
git add .

# Add all files in repository
git add -A

# Add files interactively
git add -i

# Add parts of a file
git add -p filename.txt

# Commit with message
git commit -m "Your commit message"

# Commit all modified files (skip staging)
git commit -am "Commit message"

# Amend last commit
git commit --amend -m "Updated commit message"
```

### Viewing Changes

```bash
# Show unstaged changes
git diff

# Show staged changes
git diff --staged

# Show changes in specific file
git diff filename.txt

# Show changes between commits
git diff commit1 commit2

# Show changes in last commit
git show

# Show specific commit
git show commit-hash
```

### Undoing Changes

```bash
# Discard changes in working directory
git checkout -- filename.txt

# Unstage a file (keep changes)
git reset HEAD filename.txt

# Reset to last commit (lose changes)
git reset --hard HEAD

# Reset to specific commit
git reset --hard commit-hash

# Create new commit that undoes previous commit
git revert commit-hash

# Remove untracked files
git clean -f

# Remove untracked files and directories
git clean -fd
```

---

## 📁 Working with Repositories

### Repository Structure

```
.git/                 # Git metadata (don't touch!)
├── HEAD             # Points to current branch
├── config           # Local repository settings
├── objects/         # Git objects (commits, trees, blobs)
├── refs/            # Branch and tag references
└── index            # Staging area

.gitignore           # Files to ignore
README.md            # Project documentation
```

### .gitignore File

```bash
# .gitignore example for web development

# Dependencies
node_modules/
vendor/

# Environment variables
.env
.env.local

# Build outputs
dist/
build/
*.min.js
*.min.css

# OS files
.DS_Store
Thumbs.db
*.tmp

# IDE files
.vscode/
.idea/
*.swp
*.swo

# Logs
*.log
logs/

# Database
*.sqlite
*.db

# Images (example - usually you want to track these)
# *.jpg
# *.png
# *.gif
```

### File Operations

```bash
# Remove file from Git and filesystem
git rm filename.txt

# Remove file from Git but keep in filesystem
git rm --cached filename.txt

# Move/rename file
git mv oldname.txt newname.txt

# Track empty directory (add .gitkeep)
touch empty-directory/.gitkeep
git add empty-directory/.gitkeep
```

---

## 🌿 Branching and Merging

### Working with Branches

```bash
# List branches
git branch

# List all branches (including remote)
git branch -a

# Create new branch
git branch feature-login

# Switch to branch
git checkout feature-login

# Create and switch to new branch
git checkout -b feature-signup

# Switch to previous branch
git checkout -

# Rename current branch
git branch -m new-branch-name

# Delete branch (safe)
git branch -d feature-login

# Force delete branch
git branch -D feature-login
```

### Merging Branches

```bash
# Switch to target branch (usually main)
git checkout main

# Merge feature branch
git merge feature-login

# Merge with no fast-forward (creates merge commit)
git merge --no-ff feature-login

# Abort merge if conflicts
git merge --abort

# Show merged branches
git branch --merged

# Show unmerged branches
git branch --no-merged
```

### Handling Merge Conflicts

```bash
# When conflicts occur, Git will show:
# Auto-merging filename.txt
# CONFLICT (content): Merge conflict in filename.txt
# Automatic merge failed; fix conflicts and then commit the result.

# 1. Open conflicted file and look for:
<<<<<<< HEAD
Your current branch content
=======
Incoming branch content
>>>>>>> feature-branch

# 2. Edit file to resolve conflicts
# 3. Add resolved file
git add filename.txt

# 4. Complete the merge
git commit -m "Resolve merge conflict in filename.txt"
```

---

## 🌐 Remote Repositories

### Working with Remotes

```bash
# Show remote repositories
git remote

# Show remote URLs
git remote -v

# Add remote repository
git remote add origin https://github.com/username/repository.git

# Change remote URL
git remote set-url origin https://github.com/username/new-repository.git

# Remove remote
git remote remove origin

# Rename remote
git remote rename origin upstream
```

### Pushing and Pulling

```bash
# Push to remote repository
git push origin main

# Push new branch to remote
git push -u origin feature-branch

# Push all branches
git push --all origin

# Force push (dangerous!)
git push --force origin main

# Pull changes from remote
git pull origin main

# Pull with rebase
git pull --rebase origin main

# Fetch changes without merging
git fetch origin

# Fetch all remotes
git fetch --all
```

### Tracking Branches

```bash
# Set upstream branch
git branch --set-upstream-to=origin/main

# Push and set upstream
git push -u origin feature-branch

# Show tracking relationships
git branch -vv

# Checkout remote branch
git checkout -b local-branch origin/remote-branch
```

---

## 🐙 GitHub Basics

### Repository Management

```bash
# Clone GitHub repository
git clone https://github.com/username/repository.git

# Fork workflow
# 1. Fork repository on GitHub
# 2. Clone your fork
git clone https://github.com/yourusername/repository.git

# 3. Add original repository as upstream
git remote add upstream https://github.com/originaluser/repository.git

# 4. Keep your fork updated
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

### GitHub Authentication

**HTTPS with Personal Access Token:**
```bash
# Generate token at: https://github.com/settings/tokens
# Use token as password when prompted
git push origin main
```

**SSH Key Setup:**
```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Add to SSH agent
ssh-add ~/.ssh/id_ed25519

# Copy public key to GitHub
cat ~/.ssh/id_ed25519.pub
# Paste at: https://github.com/settings/keys

# Test connection
ssh -T git@github.com
```

### Pull Requests Workflow

```bash
# 1. Create feature branch
git checkout -b add-new-feature

# 2. Make changes and commit
git add .
git commit -m "Add new feature"

# 3. Push branch to GitHub
git push origin add-new-feature

# 4. Create Pull Request on GitHub
# 5. After review and merge, cleanup
git checkout main
git pull origin main
git branch -d add-new-feature
```

---

## ✅ Best Practices

### Commit Message Guidelines

```bash
# Good commit messages:
git commit -m "Add user authentication system"
git commit -m "Fix navigation bug on mobile devices"
git commit -m "Update README with installation instructions"
git commit -m "Refactor user service for better performance"

# Bad commit messages:
git commit -m "Fix"
git commit -m "Changes"
git commit -m "Work in progress"
git commit -m "asdf"
```

### Commit Message Format

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

**Examples:**
```bash
git commit -m "feat(auth): add password reset functionality"
git commit -m "fix(ui): resolve button alignment issue"
git commit -m "docs(readme): update installation instructions"
```

### Branching Strategy

```bash
# Feature branch workflow
main                    # Stable, production-ready code
├── develop            # Integration branch (optional)
├── feature/user-auth  # New features
├── feature/payment    # Another feature
├── hotfix/security    # Critical fixes
└── release/v1.2       # Release preparation
```

### Repository Organization

```
project/
├── .git/              # Git metadata
├── .gitignore         # Ignored files
├── README.md          # Project documentation
├── LICENSE            # License file
├── CHANGELOG.md       # Change history
├── CONTRIBUTING.md    # Contribution guidelines
├── src/               # Source code
├── tests/             # Test files
├── docs/              # Documentation
└── scripts/           # Build/deployment scripts
```

---

## 🔄 Common Workflows

### Daily Development Workflow

```bash
# Start of day
git checkout main
git pull origin main

# Create feature branch
git checkout -b feature/new-component

# Work on feature
# ... make changes ...
git add .
git commit -m "feat: add new component structure"

# ... more changes ...
git add .
git commit -m "feat: implement component logic"

# Push to remote
git push -u origin feature/new-component

# Create Pull Request on GitHub

# After PR is merged
git checkout main
git pull origin main
git branch -d feature/new-component
```

### Collaboration Workflow

```bash
# Team member workflow
# 1. Pull latest changes
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature/user-profile

# 3. Work and commit regularly
git add .
git commit -m "Add user profile form"

# 4. Keep branch updated
git checkout main
git pull origin main
git checkout feature/user-profile
git merge main  # or git rebase main

# 5. Push and create PR
git push origin feature/user-profile
```

### Release Workflow

```bash
# Prepare release
git checkout main
git pull origin main

# Create release branch
git checkout -b release/v1.2.0

# Make release preparations
# Update version numbers, changelog, etc.
git commit -m "chore: prepare v1.2.0 release"

# Push release branch
git push origin release/v1.2.0

# Create Pull Request for release
# After approval, merge to main

# Tag the release
git checkout main
git pull origin main
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin v1.2.0

# Merge back to develop (if using)
git checkout develop
git merge main
git push origin develop
```

### Hotfix Workflow

```bash
# Critical bug in production
git checkout main
git pull origin main

# Create hotfix branch
git checkout -b hotfix/security-patch

# Fix the issue
git add .
git commit -m "fix: patch security vulnerability"

# Push and create emergency PR
git push origin hotfix/security-patch

# After merge and deployment
git checkout main
git pull origin main
git tag -a v1.2.1 -m "Hotfix version 1.2.1"
git push origin v1.2.1
```

---

## 📚 Quick Reference

### Essential Git Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `git init` | Initialize repository | `git init` |
| `git clone` | Copy repository | `git clone <url>` |
| `git add` | Stage changes | `git add .` |
| `git commit` | Save changes | `git commit -m "message"` |
| `git push` | Upload changes | `git push origin main` |
| `git pull` | Download changes | `git pull origin main` |
| `git status` | Check status | `git status` |
| `git log` | View history | `git log --oneline` |
| `git branch` | Manage branches | `git branch feature` |
| `git checkout` | Switch branches | `git checkout main` |
| `git merge` | Combine branches | `git merge feature` |

### Git Aliases (Optional)

```bash
# Add useful aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'

# Usage
git st          # instead of git status
git co main     # instead of git checkout main
git ci -m "msg" # instead of git commit -m "msg"
```

### Best Practices Summary:
1. Commit early and often with meaningful messages
2. Use branches for features and experiments
3. Keep main branch stable and deployable
4. Review code through Pull Requests
5. Use .gitignore to exclude unnecessary files
6. Write good commit messages
7. Keep commits focused on single changes
8. Regularly sync with remote repositories

### Next Steps:
1. Learn advanced Git commands (rebase, cherry-pick, bisect)
2. Explore Git hooks for automation
3. Study different branching strategies
4. Learn about Git workflows (GitFlow, GitHub Flow)
5. Practice collaboration on open source projects

This Git guide covers the fundamentals of version control that every developer should know. Practice these commands regularly to become proficient with Git!
