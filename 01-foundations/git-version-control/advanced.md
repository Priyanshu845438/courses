
# 🚀 Advanced Git

## Table of Contents
- [Git Internals](#git-internals)
- [Custom Git Commands](#custom-git-commands)
- [Advanced Configuration](#advanced-configuration)
- [Git Server Administration](#git-server-administration)
- [Performance Optimization](#performance-optimization)
- [Advanced Workflows](#advanced-workflows)
- [Git at Scale](#git-at-scale)
- [Troubleshooting and Recovery](#troubleshooting-and-recovery)

---

## 🔧 Git Internals

### Understanding Git Objects

```bash
# Explore .git directory structure
ls -la .git/

# Show object type
git cat-file -t HEAD

# Show object content
git cat-file -p HEAD

# Show object size
git cat-file -s HEAD

# Create a blob object manually
echo "Hello World" | git hash-object -w --stdin

# Show all objects
find .git/objects -type f | sed 's|.git/objects/||' | sed 's|/||'

# Verify object integrity
git fsck --full
```

### Plumbing vs Porcelain Commands

```bash
# Plumbing commands (low-level)
git hash-object          # Create objects
git cat-file             # Examine objects
git update-index         # Manipulate index
git write-tree           # Create tree objects
git commit-tree          # Create commit objects
git update-ref           # Update references

# Porcelain commands (high-level)
git add                  # Uses update-index
git commit               # Uses write-tree and commit-tree
git checkout             # Uses multiple plumbing commands
git merge                # Complex operation using multiple commands
```

### Manual Commit Creation

```bash
# Create a commit manually using plumbing commands
echo "Hello World" > hello.txt
git hash-object -w hello.txt  # Returns: abc123...

git update-index --add --cacheinfo 100644 abc123... hello.txt
git write-tree  # Returns: def456...

echo "Initial commit" | git commit-tree def456...  # Returns: ghi789...
git update-ref refs/heads/main ghi789...
```

---

## 🛠 Custom Git Commands

### Creating Git Aliases for Complex Operations

```bash
# Complex log alias
git config --global alias.superlog '!f() { 
    git log --graph --abbrev-commit --decorate --format=format:"%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)" --all; 
}; f'

# Find commits that introduced a bug
git config --global alias.find-bug '!f() { 
    git log -S"$1" --source --all --oneline; 
}; f'

# Clean up merged branches
git config --global alias.cleanup-branches '!f() { 
    git branch --merged | grep -v "*\\|main\\|master\\|develop" | xargs -n 1 git branch -d; 
}; f'

# Interactive rebase with specific number of commits
git config --global alias.reb '!f() { 
    git rebase -i HEAD~"$1"; 
}; f'
```

### Shell Script Git Commands

```bash
#!/bin/bash
# ~/.local/bin/git-feature (make executable)
# Usage: git feature start feature-name
# Usage: git feature finish

case "$1" in
    start)
        if [ -z "$2" ]; then
            echo "Feature name required"
            exit 1
        fi
        git checkout main
        git pull origin main
        git checkout -b "feature/$2"
        echo "Started feature branch: feature/$2"
        ;;
    finish)
        current_branch=$(git branch --show-current)
        if [[ ! $current_branch == feature/* ]]; then
            echo "Not on a feature branch"
            exit 1
        fi
        git checkout main
        git pull origin main
        git branch -d "$current_branch"
        echo "Finished feature branch: $current_branch"
        ;;
    *)
        echo "Usage: git feature {start|finish} [feature-name]"
        exit 1
        ;;
esac
```

### Advanced Git Scripts

```bash
#!/bin/bash
# git-stats - Show repository statistics

echo "=== Repository Statistics ==="
echo

echo "📊 Commit Statistics:"
echo "Total commits: $(git rev-list --all --count)"
echo "Authors: $(git shortlog -sn | wc -l)"
echo "Branches: $(git branch -a | wc -l)"
echo "Tags: $(git tag | wc -l)"
echo

echo "👥 Top Contributors:"
git shortlog -sn | head -10
echo

echo "📅 Recent Activity:"
git log --oneline --since="1 week ago" | head -10
echo

echo "📁 Largest Files:"
git rev-list --objects --all | \
    git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | \
    sed -n 's/^blob //p' | \
    sort --numeric-sort --key=2 | \
    tail -10 | \
    while read size sha path; do
        echo "$size bytes - $path"
    done
```

---

## ⚙️ Advanced Configuration

### Conditional Configuration

```bash
# ~/.gitconfig
[user]
    name = John Doe
    email = john@personal.com

[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work

[includeIf "gitdir:~/opensource/"]
    path = ~/.gitconfig-opensource

[includeIf "hasconfig:remote.*.url:git@github.com:company/*"]
    path = ~/.gitconfig-company
```

```bash
# ~/.gitconfig-work
[user]
    email = john.doe@company.com
    signingkey = ABC123DEF456

[commit]
    gpgsign = true

[push]
    default = simple

[pull]
    rebase = true
```

### Advanced Hooks

```bash
#!/bin/bash
# .git/hooks/pre-commit - Advanced pre-commit hook

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo -e "${YELLOW}Running pre-commit checks...${NC}"

# Check for merge conflicts
if git diff --cached --name-only | xargs grep -l "<<<<<<< HEAD\|======= \|>>>>>>> " 2>/dev/null; then
    echo -e "${RED}Error: Merge conflict markers found${NC}"
    exit 1
fi

# Check for debugging statements
if git diff --cached --name-only | grep -E '\.(js|ts|jsx|tsx)$' | xargs grep -n "console\.log\|debugger" 2>/dev/null; then
    echo -e "${RED}Error: Debugging statements found${NC}"
    exit 1
fi

# Run tests
if [ -f "package.json" ] && grep -q "test" package.json; then
    echo "Running tests..."
    npm test
    if [ $? -ne 0 ]; then
        echo -e "${RED}Tests failed${NC}"
        exit 1
    fi
fi

# Check code formatting
if command -v prettier >/dev/null 2>&1; then
    echo "Checking code formatting..."
    git diff --cached --name-only | grep -E '\.(js|ts|jsx|tsx|css|md)$' | xargs prettier --check
    if [ $? -ne 0 ]; then
        echo -e "${RED}Code formatting issues found. Run 'prettier --write .' to fix${NC}"
        exit 1
    fi
fi

echo -e "${GREEN}All checks passed!${NC}"
```

### Global Git Attributes

```bash
# ~/.gitattributes_global
*.pdf diff=astextplain
*.jpg binary
*.png binary
*.gif binary

# Custom diff for package-lock.json
package-lock.json diff=locktextconv

# Handle line endings
*.sh text eol=lf
*.bat text eol=crlf
*.ps1 text eol=crlf

# Set up in global config
git config --global core.attributesfile ~/.gitattributes_global
```

---

## 🖥 Git Server Administration

### Setting Up Git Server

```bash
# Create bare repository on server
git init --bare /var/git/project.git
chown -R git:git /var/git/project.git

# Configure hooks
cd /var/git/project.git/hooks

# Post-receive hook for automatic deployment
#!/bin/bash
# hooks/post-receive
while read oldrev newrev refname; do
    if [[ $refname == "refs/heads/main" ]]; then
        echo "Deploying to production..."
        cd /var/www/project
        git --git-dir=/var/git/project.git --work-tree=/var/www/project checkout -f main
        npm install
        npm run build
        systemctl restart project-service
        echo "Deployment complete!"
    fi
done
```

### Git Daemon Configuration

```bash
# Start git daemon
git daemon --reuseaddr --base-path=/var/git/ /var/git/

# Configure git daemon as systemd service
# /etc/systemd/system/git-daemon.service
[Unit]
Description=Git Daemon
After=network.target

[Service]
ExecStart=/usr/bin/git daemon --reuseaddr --base-path=/var/git/ /var/git/
Restart=always
User=git
Group=git

[Install]
WantedBy=multi-user.target
```

### Advanced Repository Management

```bash
# Repository maintenance
git gc --aggressive --prune=now
git repack -ad
git prune --expire=now

# Analyze repository size
git count-objects -vH

# Find large objects
git rev-list --objects --all | \
    git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | \
    awk '/^blob/ {print substr($0,6)}' | \
    sort --numeric-sort --key=2 | \
    tail -20

# Clean up large files from history
git filter-branch --tree-filter 'rm -f large-file.zip' HEAD
git for-each-ref --format='delete %(refname)' refs/original | git update-ref --stdin
git reflog expire --expire=now --all
git gc --prune=now
```

---

## 🚀 Performance Optimization

### Large Repository Optimization

```bash
# Enable partial clone for large repositories
git clone --filter=blob:none <url>
git clone --filter=tree:0 <url>  # Even more aggressive

# Sparse checkout for monorepos
git config core.sparseCheckout true
echo "frontend/*" > .git/info/sparse-checkout
echo "shared/*" >> .git/info/sparse-checkout
git read-tree -m -u HEAD

# Enable experimental features
git config feature.manyFiles true
git config core.preloadindex true
git config core.fscache true
git config gc.writeCommitGraph true
```

### Git LFS for Large Files

```bash
# Install and initialize Git LFS
git lfs install

# Track large file types
git lfs track "*.psd"
git lfs track "*.zip"
git lfs track "*.mp4"

# Check LFS status
git lfs ls-files
git lfs status

# Migrate existing large files
git lfs migrate import --include="*.zip"

# Prune old LFS objects
git lfs prune
```

### Optimizing Git Operations

```bash
# Configure Git for better performance
git config core.preloadindex true
git config core.fscache true
git config gc.auto 256

# Use multi-threaded operations
git config pack.threads 0  # Use all available cores

# Optimize pack files
git config pack.deltaCacheSize 2g
git config pack.packSizeLimit 2g
git config pack.windowMemory 2g

# Configure commit graph for faster operations
git config core.commitGraph true
git config gc.writeCommitGraph true
git commit-graph write --reachable --changed-paths
```

---

## 🏢 Git at Scale

### Monorepo Management

```bash
# Subtree strategy for monorepo
git subtree add --prefix=libs/shared https://github.com/company/shared-lib main --squash
git subtree pull --prefix=libs/shared https://github.com/company/shared-lib main --squash
git subtree push --prefix=libs/shared https://github.com/company/shared-lib main

# Workspace configuration for monorepo
# .gitmodules
[submodule "libs/ui-components"]
    path = libs/ui-components
    url = https://github.com/company/ui-components.git
    branch = main

[submodule "libs/utils"]
    path = libs/utils
    url = https://github.com/company/utils.git
    branch = main
```

### Enterprise Git Workflows

```bash
# Feature branch with integration testing
#!/bin/bash
# Automated integration workflow

feature_branch="$1"
integration_branch="integration-$(date +%Y%m%d-%H%M%S)"

# Create integration branch
git checkout main
git pull origin main
git checkout -b "$integration_branch"

# Merge feature branch
git merge --no-ff "$feature_branch"

# Run comprehensive tests
npm run test:unit
npm run test:integration
npm run test:e2e

if [ $? -eq 0 ]; then
    echo "All tests passed. Ready for production merge."
    git checkout main
    git merge --no-ff "$integration_branch"
    git push origin main
    git branch -d "$integration_branch"
else
    echo "Tests failed. Integration branch preserved for debugging."
    exit 1
fi
```

### Distributed Development

```bash
# Multiple remotes for distributed development
git remote add origin https://github.com/company/project.git
git remote add upstream https://github.com/original/project.git
git remote add fork https://github.com/yourname/project.git

# Sync workflow
git fetch --all
git checkout main
git merge upstream/main
git push origin main
git push fork main

# Cherry-pick across forks
git fetch upstream
git cherry-pick upstream/feature-branch~2..upstream/feature-branch
```

---

## 🔧 Troubleshooting and Recovery

### Data Recovery

```bash
# Recover deleted branch
git reflog  # Find the commit SHA
git checkout -b recovered-branch SHA

# Recover deleted commits
git fsck --lost-found
ls .git/lost-found/commit/
git show SHA

# Recover from corrupted repository
git fsck --full
git gc --prune=now

# Clone from multiple sources to recover
git clone --mirror broken-repo recovered-repo
cd recovered-repo
git fsck --full
```

### Advanced Debugging

```bash
# Debug Git operations
GIT_TRACE=1 git push
GIT_TRACE_PACKET=1 git fetch
GIT_TRACE_SETUP=1 git status

# Debug performance issues
GIT_TRACE_PERFORMANCE=1 git log --oneline -1000

# Analyze repository corruption
git fsck --full --strict
git cat-file -e SHA  # Check if object exists
git verify-pack -v .git/objects/pack/pack-*.idx
```

### Emergency Procedures

```bash
#!/bin/bash
# Emergency backup and recovery script

REPO_DIR="$(pwd)"
BACKUP_DIR="/tmp/git-emergency-backup-$(date +%s)"

echo "Creating emergency backup..."
mkdir -p "$BACKUP_DIR"

# Backup entire .git directory
cp -r .git "$BACKUP_DIR/"

# Backup all tracked files
git ls-files | tar -czf "$BACKUP_DIR/tracked-files.tar.gz" -T -

# Backup all refs
cp -r .git/refs "$BACKUP_DIR/"
cp .git/HEAD "$BACKUP_DIR/"

# Export all branches
git for-each-ref --format='%(refname:short)' refs/heads/ | while read branch; do
    git format-patch --output-directory="$BACKUP_DIR/patches/" --root "$branch"
done

echo "Emergency backup created at: $BACKUP_DIR"

# Recovery function
recover_repository() {
    local backup_dir="$1"
    
    if [ ! -d "$backup_dir" ]; then
        echo "Backup directory not found"
        return 1
    fi
    
    # Restore .git directory
    rm -rf .git
    cp -r "$backup_dir/.git" .
    
    # Restore working directory
    git reset --hard HEAD
    
    echo "Repository recovered from backup"
}
```

---

## 📊 Advanced Analytics and Reporting

### Custom Git Analytics

```bash
#!/bin/bash
# Advanced git analytics script

# Function to get commits by time period
commits_by_period() {
    local period="$1"
    local format="$2"
    
    git log --pretty=format:"%ad" --date="$format" --since="$period" | \
    sort | uniq -c | sort -rn
}

# Code churn analysis
code_churn() {
    git log --pretty=format: --name-only --since="1 month ago" | \
    grep -v '^$' | sort | uniq -c | sort -rn | head -20
}

# Developer velocity
developer_velocity() {
    git log --pretty=format:"%an %ad" --date=short --since="1 month ago" | \
    awk '{
        author = $1; 
        date = $2; 
        commits[author][date]++; 
        total[author]++
    } 
    END {
        for (author in total) {
            days = length(commits[author]);
            avg = total[author] / days;
            printf "%s: %.2f commits/day (%d total, %d active days)\n", 
                   author, avg, total[author], days
        }
    }'
}

# Generate comprehensive report
generate_report() {
    echo "=== Git Repository Analysis Report ==="
    echo "Generated: $(date)"
    echo
    
    echo "📈 Commits by month:"
    commits_by_period "1 year ago" "format:%Y-%m"
    echo
    
    echo "🔥 Most changed files (last month):"
    code_churn
    echo
    
    echo "⚡ Developer velocity:"
    developer_velocity
    echo
}

# Run the report
generate_report > git-analytics-report.txt
echo "Report saved to git-analytics-report.txt"
```

This advanced Git guide covers internals, server administration, performance optimization, enterprise workflows, and advanced troubleshooting techniques for professional Git usage at scale.
# 🚀 Git Version Control - Advanced

## 🏗️ Advanced Git Workflows

### Git Flow Workflow

```bash
# Initialize git flow
git flow init

# Start a new feature
git flow feature start new-login-system

# Work on feature
git add .
git commit -m "Add login form validation"

# Finish feature (merges to develop)
git flow feature finish new-login-system

# Start a release
git flow release start 1.2.0

# Finish release (merges to main and develop)
git flow release finish 1.2.0

# Create hotfix
git flow hotfix start critical-bug-fix

# Finish hotfix (merges to main and develop)
git flow hotfix finish critical-bug-fix
```

### GitHub Flow (Simplified)

```bash
# Create feature branch from main
git checkout main
git pull origin main
git checkout -b feature/user-authentication

# Work and commit
git add .
git commit -m "Implement user authentication"

# Push and create PR
git push origin feature/user-authentication
# Create Pull Request on GitHub

# After PR approval, merge and cleanup
git checkout main
git pull origin main
git branch -d feature/user-authentication
```

### Advanced Branching Strategies

```bash
# Create branch from specific commit
git checkout -b hotfix/security-patch abc123

# Create orphan branch (no history)
git checkout --orphan gh-pages

# Track remote branch
git checkout -b local-branch origin/remote-branch

# Push branch with upstream
git push -u origin feature-branch

# Rename branch
git branch -m old-name new-name

# Delete remote branch
git push origin --delete branch-name
```

## 🔧 Advanced Git Commands

### Interactive Rebase

```bash
# Interactive rebase last 5 commits
git rebase -i HEAD~5

# Rebase onto different branch
git rebase -i main

# Continue rebase after resolving conflicts
git rebase --continue

# Abort rebase
git rebase --abort

# Squash commits interactively
# In editor, change 'pick' to 'squash' for commits to combine
```

### Cherry-picking

```bash
# Cherry-pick specific commit
git cherry-pick abc123

# Cherry-pick multiple commits
git cherry-pick abc123 def456 ghi789

# Cherry-pick with edit
git cherry-pick -e abc123

# Cherry-pick without commit
git cherry-pick -n abc123
```

### Advanced Merging

```bash
# Merge with custom message
git merge --no-ff feature-branch -m "Merge feature branch"

# Merge specific files only
git checkout main
git checkout feature-branch -- file1.txt file2.txt
git commit -m "Merge specific files from feature"

# Merge with strategy
git merge -X ours feature-branch  # Prefer current branch
git merge -X theirs feature-branch  # Prefer incoming branch

# Abort merge
git merge --abort
```

### Stashing Advanced

```bash
# Stash with message
git stash save "Work in progress on feature X"

# Stash including untracked files
git stash -u

# Stash specific files
git stash -- file1.txt file2.txt

# List stashes
git stash list

# Apply specific stash
git stash apply stash@{2}

# Pop stash (apply and remove)
git stash pop

# Drop stash
git stash drop stash@{1}

# Clear all stashes
git stash clear

# Create branch from stash
git stash branch new-feature stash@{0}
```

## 🔍 Advanced Git Tools

### Bisect for Bug Hunting

```bash
# Start bisect
git bisect start

# Mark current commit as bad
git bisect bad

# Mark known good commit
git bisect good abc123

# Git will checkout middle commit
# Test and mark as good or bad
git bisect good  # or git bisect bad

# Continue until bug is found
# Reset after finding bug
git bisect reset
```

### Git Hooks

#### Pre-commit Hook
```bash
#!/bin/sh
# .git/hooks/pre-commit

# Run linter
npm run lint
if [ $? -ne 0 ]; then
    echo "Linting failed. Please fix errors before committing."
    exit 1
fi

# Run tests
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed. Please fix tests before committing."
    exit 1
fi

# Check for console.log statements
if grep -r "console.log" src/; then
    echo "Remove console.log statements before committing."
    exit 1
fi

exit 0
```

#### Post-receive Hook (Server-side)
```bash
#!/bin/sh
# .git/hooks/post-receive

# Deploy to production
while read oldrev newrev refname; do
    if [ "$refname" = "refs/heads/main" ]; then
        echo "Deploying to production..."
        cd /var/www/app
        git --git-dir=/var/www/app/.git --work-tree=/var/www/app pull origin main
        npm ci
        npm run build
        sudo systemctl restart nginx
        echo "Deployment complete!"
    fi
done
```

### Git Aliases for Productivity

```bash
# Add to ~/.gitconfig
[alias]
    # Shortcuts
    co = checkout
    br = branch
    ci = commit
    st = status
    unstage = reset HEAD --
    last = log -1 HEAD
    visual = !gitk
    
    # Logging
    lg = log --oneline --graph --decorate --all
    lga = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --all
    
    # Useful commands
    amend = commit --amend --no-edit
    fixup = commit --fixup
    squash = commit --squash
    
    # Find commits
    find = log --pretty=\"format:%Cgreen%H %Cblue%s\" --name-status --grep
    
    # Undo
    undo = reset --soft HEAD~1
    undohard = reset --hard HEAD~1
    
    # Stash
    save = stash save -u
    wip = commit -am "WIP"
    
    # Cleanup
    cleanup = "!git branch --merged | grep -v '\\*\\|master\\|main\\|develop' | xargs -n 1 git branch -d"
```

## 📊 Git Statistics and Analysis

### Repository Analysis

```bash
# Show contribution statistics
git shortlog -sn --all

# Show file change statistics
git log --stat

# Show commit count by author
git shortlog -sn --since="2023-01-01" --until="2023-12-31"

# Show lines of code by author
git log --author="John Doe" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }'

# Show commits by date
git log --since="2023-01-01" --until="2023-12-31" --pretty=format:"%h %ad %s" --date=short

# Show most changed files
git log --name-only --pretty=format: | sort | uniq -c | sort -rg | head -10

# Show commit frequency by hour
git log --pretty=format:%cd --date=format:%H | sort | uniq -c
```

### Repository Health Check

```bash
# Check repository integrity
git fsck --full

# Show repository size
git count-objects -vH

# Clean up repository
git gc --aggressive --prune=now

# Remove untracked files
git clean -fdx

# Show large files in history
git rev-list --objects --all | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | sed -n 's/^blob //p' | sort --numeric-sort --key=2 | tail -20
```

## 🔒 Git Security Best Practices

### Commit Signing

```bash
# Generate GPG key
gpg --full-generate-key

# List keys
gpg --list-secret-keys --keyid-format=long

# Configure Git to use GPG
git config --global user.signingkey [KEY-ID]
git config --global commit.gpgsign true

# Sign commits
git commit -S -m "Signed commit"

# Verify signatures
git log --show-signature
```

### Protecting Sensitive Data

```bash
# Remove sensitive files from history
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch secrets.txt' --prune-empty --tag-name-filter cat -- --all

# Using git-secrets to prevent leaks
git secrets --register-aws
git secrets --install
git secrets --scan
```

### Security Configuration

```bash
# Enable fsck on receive
git config receive.fsckObjects true

# Reject non-fast-forward pushes
git config receive.denyNonFastForwards true

# Reject deletes
git config receive.denyDeletes true

# Require signed commits
git config receive.advertisePushOptions true
```

## 🚀 Git for CI/CD

### GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0  # Shallow clones should be disabled for better analysis
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linter
      run: npm run lint
    
    - name: Run tests
      run: npm run test:coverage
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
    
    - name: Build application
      run: npm run build
    
    - name: Deploy to staging
      if: github.ref == 'refs/heads/develop'
      run: |
        echo "Deploying to staging..."
        # Add deployment commands here
    
    - name: Deploy to production
      if: github.ref == 'refs/heads/main'
      run: |
        echo "Deploying to production..."
        # Add deployment commands here
```

### GitLab CI/CD Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - lint
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "18"

before_script:
  - apt-get update -qq && apt-get install -y -qq git curl libelf1
  - curl -fsSL https://deb.nodesource.com/setup_${NODE_VERSION}.x | bash -
  - apt-get install -y nodejs

lint:
  stage: lint
  script:
    - npm ci
    - npm run lint
  only:
    - merge_requests
    - main
    - develop

test:
  stage: test
  script:
    - npm ci
    - npm run test:coverage
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
  only:
    - merge_requests
    - main
    - develop

build:
  stage: build
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour
  only:
    - main
    - develop

deploy_staging:
  stage: deploy
  script:
    - echo "Deploying to staging..."
    # Add deployment commands
  environment:
    name: staging
  only:
    - develop

deploy_production:
  stage: deploy
  script:
    - echo "Deploying to production..."
    # Add deployment commands
  environment:
    name: production
  only:
    - main
  when: manual
```

## 🔧 Git Troubleshooting

### Common Issues and Solutions

```bash
# Large repository issues
git clone --depth 1 --shallow-submodules <url>  # Shallow clone
git fetch --unshallow  # Convert to full clone

# Corrupted repository
git fsck --full
git gc --aggressive

# Recover lost commits
git reflog
git checkout <commit-hash>

# Remove large files from history
git filter-branch --tree-filter 'rm -f large-file.zip' HEAD
# Or use BFG Repo-Cleaner
bfg --strip-blobs-bigger-than 10M

# Fix detached HEAD
git checkout main
git branch temp-branch <commit-hash>
git merge temp-branch

# Reset to remote state
git fetch origin
git reset --hard origin/main

# Recover deleted branch
git reflog
git checkout -b recovered-branch <commit-hash>
```

## 📚 Advanced Git Concepts

### Git Internals

```bash
# Show Git objects
git cat-file -p <hash>

# Show object type
git cat-file -t <hash>

# Show tree structure
git ls-tree HEAD

# Show Git directory structure
ls -la .git/

# Manual commit creation
git hash-object -w file.txt
git update-index --add --cacheinfo 100644 <hash> file.txt
git write-tree
git commit-tree <tree-hash> -m "Manual commit"
```

### Custom Git Commands

```bash
# Create custom Git command
# File: /usr/local/bin/git-summary
#!/bin/bash
echo "Repository Summary"
echo "=================="
echo "Branch: $(git branch --show-current)"
echo "Commits: $(git rev-list --count HEAD)"
echo "Authors: $(git shortlog -sn | wc -l)"
echo "Files: $(git ls-files | wc -l)"
echo "Size: $(du -sh .git | cut -f1)"

# Make executable
chmod +x /usr/local/bin/git-summary

# Use as: git summary
```

This advanced Git guide covers enterprise-level workflows, security practices, CI/CD integration, and troubleshooting techniques essential for professional development environments.

## 📚 Summary

### Advanced Git Mastery:
- **Workflow Management**: Git Flow, GitHub Flow, and custom workflows
- **Advanced Commands**: Interactive rebase, cherry-picking, bisect
- **Automation**: Hooks, aliases, and custom commands
- **Security**: Commit signing, sensitive data protection
- **CI/CD Integration**: GitHub Actions, GitLab CI/CD
- **Troubleshooting**: Repository maintenance and recovery
- **Internals**: Understanding Git's internal structure

### Best Practices:
1. **Use appropriate branching strategies** for your team size
2. **Implement commit signing** for security
3. **Set up automated testing** with Git hooks
4. **Use semantic commit messages** for better history
5. **Regularly clean up branches** and stashes
6. **Implement proper CI/CD pipelines**
7. **Monitor repository health** with regular maintenance

---

**Next:** Proceed to Phase 2: Frontend Development
