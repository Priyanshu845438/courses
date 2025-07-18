
# 🚀 Intermediate Git

## Table of Contents
- [Advanced Branching Strategies](#advanced-branching-strategies)
- [Git Rebase and History Rewriting](#git-rebase-and-history-rewriting)
- [Git Hooks](#git-hooks)
- [Advanced Merging](#advanced-merging)
- [Git Stash](#git-stash)
- [Collaborative Workflows](#collaborative-workflows)
- [Git Submodules](#git-submodules)
- [Debugging with Git](#debugging-with-git)

---

## 🌿 Advanced Branching Strategies

### Git Flow Workflow

```bash
# Initialize git flow
git flow init

# Start a new feature
git flow feature start user-authentication

# Work on feature
git add .
git commit -m "Add login form"
git commit -m "Add password validation"

# Finish feature
git flow feature finish user-authentication

# Start a release
git flow release start v1.2.0

# Finish release
git flow release finish v1.2.0

# Start a hotfix
git flow hotfix start critical-bug
```

### GitHub Flow (Simplified)

```bash
# Create feature branch from main
git checkout main
git pull origin main
git checkout -b feature/new-dashboard

# Make changes and commit
git add .
git commit -m "Add new dashboard layout"

# Push and create PR
git push -u origin feature/new-dashboard
# Create Pull Request on GitHub

# After PR approval, merge and cleanup
git checkout main
git pull origin main
git branch -d feature/new-dashboard
```

### Feature Flag Workflow

```bash
# Create feature branch
git checkout -b feature/experimental-ui

# Commit with feature flag
git add .
git commit -m "Add experimental UI behind feature flag"

# Merge to main (feature disabled by default)
git checkout main
git merge feature/experimental-ui

# Enable feature in production when ready
git commit -m "Enable experimental UI feature flag"
```

---

## 🔄 Git Rebase and History Rewriting

### Interactive Rebase

```bash
# Interactive rebase last 3 commits
git rebase -i HEAD~3

# In the editor, you can:
# pick abc123 First commit
# squash def456 Second commit (will be combined with first)
# edit ghi789 Third commit (will pause for editing)

# Common rebase commands:
pick    # Keep commit as is
squash  # Combine with previous commit
edit    # Pause to modify commit
drop    # Remove commit entirely
reword  # Change commit message
```

### Cleaning Up History

```bash
# Squash last 3 commits into one
git reset --soft HEAD~3
git commit -m "Combined feature implementation"

# Amend last commit message
git commit --amend -m "Better commit message"

# Amend last commit with new changes
git add forgotten-file.js
git commit --amend --no-edit

# Split a commit
git reset HEAD~1
git add file1.js
git commit -m "First part of changes"
git add file2.js
git commit -m "Second part of changes"
```

### Rebase vs Merge

```bash
# Merge creates a merge commit
git checkout main
git merge feature-branch
# Creates: A---B---C---M (merge commit)
#               \     /
#                D---E

# Rebase replays commits
git checkout feature-branch
git rebase main
# Creates: A---B---C---D'---E'
```

---

## 🎣 Git Hooks

### Client-Side Hooks

```bash
# Pre-commit hook (.git/hooks/pre-commit)
#!/bin/sh
# Run linting before commit
npm run lint
if [ $? -ne 0 ]; then
    echo "Linting failed. Please fix errors before committing."
    exit 1
fi

# Run tests
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed. Please fix before committing."
    exit 1
fi
```

```bash
# Commit message hook (.git/hooks/commit-msg)
#!/bin/sh
# Validate commit message format
commit_regex='^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .{1,50}'

if ! grep -qE "$commit_regex" "$1"; then
    echo "Invalid commit message format!"
    echo "Format: type(scope): description"
    echo "Example: feat(auth): add user login"
    exit 1
fi
```

### Server-Side Hooks

```bash
# Pre-receive hook (on server)
#!/bin/sh
# Prevent force pushes to main
while read oldrev newrev refname; do
    if [ "$refname" = "refs/heads/main" ]; then
        if [ "$oldrev" != "0000000000000000000000000000000000000000" ]; then
            # Check if it's a force push
            if ! git merge-base --is-ancestor "$oldrev" "$newrev"; then
                echo "Force push to main branch is not allowed!"
                exit 1
            fi
        fi
    fi
done
```

### Automated Hooks with Husky

```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS",
      "pre-push": "npm test"
    }
  },
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": ["eslint --fix", "git add"],
    "*.{css,scss}": ["stylelint --fix", "git add"]
  }
}
```

---

## 🔀 Advanced Merging

### Merge Strategies

```bash
# Fast-forward merge (default when possible)
git merge feature-branch

# No fast-forward (always create merge commit)
git merge --no-ff feature-branch

# Squash merge (combine all commits into one)
git merge --squash feature-branch
git commit -m "Feature: Complete user authentication system"

# Recursive merge with strategy
git merge -X theirs feature-branch  # Prefer their changes
git merge -X ours feature-branch    # Prefer our changes
```

### Resolving Complex Conflicts

```bash
# Use mergetool for complex conflicts
git mergetool

# Manual conflict resolution
git status  # See conflicted files

# Edit conflicted file
<<<<<<< HEAD
console.log('Our version');
=======
console.log('Their version');
>>>>>>> feature-branch

# Choose resolution and mark as resolved
git add conflicted-file.js
git commit
```

### Cherry-picking

```bash
# Apply specific commit to current branch
git cherry-pick abc123

# Cherry-pick multiple commits
git cherry-pick abc123 def456 ghi789

# Cherry-pick with edit
git cherry-pick -e abc123

# Cherry-pick without committing
git cherry-pick -n abc123
```

---

## 📚 Git Stash

### Basic Stashing

```bash
# Stash current changes
git stash

# Stash with message
git stash save "Work in progress on user profile"

# List stashes
git stash list

# Apply latest stash
git stash apply

# Apply specific stash
git stash apply stash@{2}

# Pop stash (apply and remove)
git stash pop

# Drop stash
git stash drop stash@{1}
```

### Advanced Stashing

```bash
# Stash only staged changes
git stash --staged

# Stash including untracked files
git stash -u

# Stash specific files
git stash push -m "Partial work" -- file1.js file2.css

# Create branch from stash
git stash branch feature-branch stash@{1}

# Show stash contents
git stash show -p stash@{0}
```

---

## 👥 Collaborative Workflows

### Code Review Process

```bash
# Create feature branch
git checkout -b feature/user-settings

# Make changes and push
git push -u origin feature/user-settings

# Create pull request (GitHub/GitLab)
# After review, address feedback:

# Make changes based on review
git add .
git commit -m "Address review feedback"
git push

# Squash commits before merge
git rebase -i HEAD~3
```

### Handling Large Teams

```bash
# Fetch all remote branches
git fetch --all

# Track remote branch
git checkout -b feature/new-api origin/feature/new-api

# Update fork from upstream
git remote add upstream https://github.com/original/repo.git
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

### Release Management

```bash
# Tag releases
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin v1.2.0

# List tags
git tag -l

# Show tag info
git show v1.2.0

# Delete tag
git tag -d v1.2.0
git push origin --delete v1.2.0

# Create release branch
git checkout -b release/v1.2.0
git push -u origin release/v1.2.0
```

---

## 📦 Git Submodules

### Adding Submodules

```bash
# Add submodule
git submodule add https://github.com/user/shared-library.git lib/shared

# Initialize submodules after cloning
git submodule init
git submodule update

# Clone with submodules
git clone --recursive https://github.com/user/main-project.git

# Update submodule to latest
cd lib/shared
git pull origin main
cd ../..
git add lib/shared
git commit -m "Update shared library"
```

### Managing Submodules

```bash
# Update all submodules
git submodule update --remote

# Update specific submodule
git submodule update --remote lib/shared

# Remove submodule
git submodule deinit lib/shared
git rm lib/shared
rm -rf .git/modules/lib/shared
```

---

## 🐛 Debugging with Git

### Git Bisect

```bash
# Start bisect session
git bisect start

# Mark current commit as bad
git bisect bad

# Mark known good commit
git bisect good v1.0.0

# Git will checkout middle commit
# Test and mark as good or bad
git bisect good  # or git bisect bad

# Continue until bug is found
# Reset when done
git bisect reset
```

### Automated Bisect

```bash
# Automated bisect with test script
git bisect start HEAD v1.0.0
git bisect run npm test

# Or with custom script
git bisect run ./test-script.sh
```

### Git Blame and Log

```bash
# Show who changed each line
git blame filename.js

# Show blame for specific lines
git blame -L 10,20 filename.js

# Follow file renames
git log --follow filename.js

# Show commits that touched specific function
git log -S "functionName" --source --all

# Find commits that changed specific line
git log -L 15,25:filename.js
```

### Searching History

```bash
# Search commit messages
git log --grep="fix"

# Search code changes
git log -S "searchString"

# Search by author
git log --author="John Doe"

# Search by date
git log --since="2023-01-01" --until="2023-12-31"

# Show commits between branches
git log main..feature-branch

# Show changes in specific file over time
git log -p -- filename.js
```

---

## 🔧 Git Configuration and Aliases

### Advanced Configuration

```bash
# Global configurations
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global core.editor "code --wait"
git config --global merge.tool vimdiff
git config --global push.default simple
git config --global pull.rebase true

# Conditional includes (different configs for work/personal)
# ~/.gitconfig
[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work
[includeIf "gitdir:~/personal/"]
    path = ~/.gitconfig-personal
```

### Useful Aliases

```bash
# Productivity aliases
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'

# Advanced aliases
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
git config --global alias.cleanup "!git branch --merged | grep -v '*\\|main\\|develop' | xargs -n 1 git branch -d"
git config --global alias.contributors "shortlog -sn"
git config --global alias.amend "commit --amend --no-edit"
```

---

## 📊 Git Statistics and Analysis

### Repository Analysis

```bash
# Show repository statistics
git log --shortstat --since="1 month ago" | grep -E "(files? changed|insertions?|deletions?)" | awk '{files+=$1; inserted+=$4; deleted+=$6} END {print "Files changed:", files, "Insertions:", inserted, "Deletions:", deleted}'

# Show top contributors
git shortlog -sn

# Show commits per day
git log --pretty=format:"%ad" --date=short | sort | uniq -c

# Show file change frequency
git log --name-only --pretty=format: | sort | uniq -c | sort -rg

# Show largest files in repository
git rev-list --objects --all | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | sed -n 's/^blob //p' | sort --numeric-sort --key=2 | tail -10
```

---

## 🔐 Security and Best Practices

### Signing Commits

```bash
# Generate GPG key
gpg --gen-key

# List GPG keys
gpg --list-secret-keys --keyid-format LONG

# Configure Git to use GPG key
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true

# Sign individual commit
git commit -S -m "Signed commit"

# Verify signatures
git log --show-signature
```

### Protecting Sensitive Data

```bash
# Remove sensitive file from history
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch secrets.txt' --prune-empty --tag-name-filter cat -- --all

# Using BFG Repo-Cleaner (faster alternative)
bfg --delete-files secrets.txt
git reflog expire --expire=now --all && git gc --prune=now --aggressive

# Prevent future leaks with .gitignore
echo "*.env" >> .gitignore
echo "secrets.txt" >> .gitignore
git add .gitignore
git commit -m "Add gitignore for sensitive files"
```

This intermediate Git guide covers advanced workflows, history management, collaboration patterns, and debugging techniques essential for professional development teams.
