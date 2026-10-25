# GitHub Workflow Guide

## Overview

This guide explains the recommended GitHub workflow for Socially development.

## Current Status

**NOT INITIALIZED:** The project is not currently a Git repository.

**Target Repository:** https://github.com/mrhasibuldev/x-media.git

**Repository Status:** Empty (no commits, no files)

---

## Git Initialization

### Step 1: Initialize Git Repository

```bash
cd /path/to/project
git init
```

**What this does:**
- Creates `.git` directory
- Initializes Git repository
- Enables version control

---

### Step 2: Create .gitignore

**Current Status:** `.gitignore` already exists and is properly configured.

**Verified Exclusions:**
- `node_modules/` - Dependencies
- `.next/` - Build output
- `.env` - Environment files
- `*.log` - Log files
- `.DS_Store` - macOS files

**Note:** Prisma migrations (`prisma/migrations/`) are NOT excluded and should be committed.

---

### Step 3: Create Initial Commit

```bash
git add .
git commit -m "Initial commit: Socially social media application"
```

**What this does:**
- Stages all files
- Creates initial commit
- Establishes baseline

---

### Step 4: Add Remote Repository

```bash
git remote add origin https://github.com/mrhasibuldev/x-media.git
```

**What this does:**
- Links local repository to GitHub
- Sets remote name to "origin"

---

### Step 5: Push to GitHub

```bash
git branch -M main
git push -u origin main
```

**What this does:**
- Renames current branch to "main"
- Pushes to GitHub
- Sets upstream tracking

---

## Branching Strategy

### Recommended Strategy: GitHub Flow

**Main Branch:**
- Always production-ready
- Protected (require pull requests)
- No direct commits allowed

**Feature Branches:**
- Created from main
- Named descriptively (e.g., `feature/add-search`)
- Merged via pull request
- Deleted after merge

**Branch Naming Convention:**
- `feature/feature-name` - New features
- `bugfix/bug-description` - Bug fixes
- `hotfix/critical-fix` - Urgent production fixes
- `docs/documentation-update` - Documentation changes

---

### Branch Creation

```bash
# Create and switch to new branch
git checkout -b feature/add-search

# Or create then switch
git branch feature/add-search
git checkout feature/add-search
```

---

### Branch Merging

**Via Pull Request (Recommended):**
1. Push feature branch to GitHub
2. Create pull request on GitHub
3. Request review (if team)
4. Merge after approval
5. Delete feature branch

**Via Command Line (Not Recommended for Main):**
```bash
git checkout main
git merge feature/add-search
git push
```

---

## Commit Conventions

### Commit Message Format

**Recommended Format:**
```
type(scope): subject

body

footer
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation change
- `style` - Code style change (formatting, etc.)
- `refactor` - Code refactoring
- `test` - Adding or updating tests
- `chore` - Maintenance tasks

**Examples:**
```
feat(auth): add two-factor authentication

- Implement 2FA flow
- Add verification code input
- Update user settings page

Closes #123
```

```
fix(post): prevent duplicate likes

- Add unique constraint to Like table
- Update toggleLike to handle constraint
- Add error handling for duplicates

Fixes #456
```

---

### Commit Best Practices

**Do:**
- Use present tense ("add" not "added")
- Use imperative mood ("add feature" not "added feature")
- Capitalize subject line
- Limit subject line to 50 characters
- Wrap body at 72 characters
- Reference related issues

**Don't:**
- Don't use vague messages ("update", "fix stuff")
- Don't combine unrelated changes
- Don't commit half-done work
- Don't commit debugging code

---

## Pull Request Process

### Creating a Pull Request

1. **Update Branch:**
   ```bash
   git checkout main
   git pull origin main
   git checkout feature/your-feature
   git rebase main
   ```

2. **Push Changes:**
   ```bash
   git push origin feature/your-feature
   ```

3. **Create PR on GitHub:**
   - Go to repository on GitHub
   - Click "New Pull Request"
   - Select feature branch
   - Describe changes
   - Reference related issues
   - Request reviewers (if applicable)

4. **Review and Merge:**
   - Address review feedback
   - Update PR with new commits
   - Merge when approved
   - Delete feature branch

---

### Pull Request Template

**Recommended PR Description:**

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Tested locally
- [ ] Added/updated tests
- [ ] Manual testing steps

## Checklist
- [ ] Code follows project style
- [ ] Self-review completed
- [ ] Commented complex code
- [ ] Updated documentation
- [ ] No merge conflicts
- [ ] No console errors
- [ ] Environment variables documented

## Related Issues
Closes #123
```

---

## Code Review Guidelines

### For Reviewers

**What to Check:**
- Code quality and style
- Bug potential
- Security issues
- Performance concerns
- Test coverage
- Documentation updates
- Breaking changes

**How to Review:**
1. Read the diff carefully
2. Test changes locally if possible
3. Ask questions about unclear code
4. Suggest improvements
5. Approve or request changes

### For Authors

**How to Handle Feedback:**
- Respond to all comments
- Make requested changes
- Explain your reasoning if you disagree
- Update PR description if scope changes
- Re-request review after changes

---

## Main Branch Protection

### Recommended Protection Rules

**Branch Protection Settings:**
1. Require pull request before merging
2. Require at least 1 approval (if team)
3. Require status checks to pass (when tests exist)
4. Require branches to be up to date
5. Do not allow bypassing settings

**How to Configure:**
1. Go to repository on GitHub
2. Settings → Branches
3. Add rule for "main" branch
4. Configure restrictions

---

## Continuous Integration (CI)

### Current Status

**NOT IMPLEMENTED:** No CI/CD workflows exist.

### Recommended GitHub Actions

**Workflow 1: Lint and Type Check**

```yaml
name: Lint and Type Check

on:
  push:
    branches: [main, feature/*]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run lint
      - run: npx tsc --noEmit
```

**Workflow 2: Build Test**

```yaml
name: Build Test

on:
  push:
    branches: [main, feature/*]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run build
```

**Workflow 3: Dependency Audit**

```yaml
name: Dependency Audit

on:
  schedule:
    - cron: '0 0 * * 0' # Weekly
  push:
    branches: [main]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm audit
```

---

## Release Workflow

### Version Tagging

**Recommended Format:**
- `v1.0.0` - Initial stable release
- `v1.0.1` - Patch release (bug fixes)
- `v1.1.0` - Minor release (new features)
- `v2.0.0` - Major release (breaking changes)

**Creating a Tag:**
```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0
```

**Creating a Release on GitHub:**
1. Go to repository on GitHub
2. Click "Releases"
3. Click "Create a new release"
4. Choose tag
5. Describe changes
6. Publish release

---

## Forking and Contributing

### For External Contributors

1. **Fork Repository:**
   - Click "Fork" on GitHub
   - Clone fork locally

2. **Create Feature Branch:**
   ```bash
   git checkout -b feature/your-feature
   ```

3. **Make Changes:**
   - Edit code
   - Commit changes
   - Push to fork

4. **Create Pull Request:**
   - Go to original repository
   - Click "New Pull Request"
   - Select your fork and branch
   - Describe changes

---

## Git Best Practices

### Do

- Commit frequently with meaningful messages
- Pull before pushing to avoid conflicts
- Use branches for features
- Write good commit messages
- Review your changes before committing
- Keep commits atomic (one logical change per commit)
- Delete merged branches

### Don't

- Don't commit large binary files
- Don't commit sensitive data (API keys, passwords)
- Don't commit to main directly
- Don't force push unless absolutely necessary
- Don't leave merge conflicts unresolved
- Don't ignore merge conflicts
- Don't commit half-done work

---

## Resolving Merge Conflicts

### When Conflicts Occur

```bash
# 1. Pull latest changes
git pull origin main

# 2. Resolve conflicts in affected files
# Edit files to resolve conflicts

# 3. Stage resolved files
git add <resolved-files>

# 4. Complete merge
git commit

# 5. Push changes
git push
```

### Conflict Markers

```
<<<<<<< HEAD
Your changes
=======
Their changes
>>>>>>> branch-name
```

**Resolution:**
- Choose which version to keep
- Or combine both versions
- Remove conflict markers
- Stage and commit

---

## Git Commands Reference

### Common Commands

```bash
# Status
git status

# Add files
git add .
git add <file>

# Commit
git commit -m "message"

# Push
git push
git push origin <branch>

# Pull
git pull
git pull origin main

# Branch
git branch
git branch <branch-name>
git checkout -b <branch-name>
git checkout <branch-name>
git branch -d <branch-name>

# Merge
git merge <branch-name>

# Rebase
git rebase main

# Stash
git stash
git stash pop

# Log
git log
git log --oneline
git log --graph

# Reset
git reset --soft HEAD~1
git reset --hard HEAD~1

# Remote
git remote -v
git remote add origin <url>
git remote remove origin
```

---

## Troubleshooting

### Common Issues

**Issue: "Updates were rejected because the remote contains work"**

**Solution:**
```bash
git pull origin main
# Resolve conflicts
git push
```

**Issue: "Failed to push some refs"**

**Solution:**
```bash
git pull --rebase origin main
git push
```

**Issue: "Branch has no upstream branch"**

**Solution:**
```bash
git push -u origin <branch-name>
```

---

## Next Steps

1. Initialize Git repository
2. Create initial commit
3. Push to GitHub
4. Configure branch protection
5. Set up CI/CD workflows
6. Establish commit conventions
7. Train team on workflow

## Resources

- **Git Documentation:** https://git-scm.com/doc
- **GitHub Flow:** https://guides.github.com/introduction/flow/
- **Conventional Commits:** https://www.conventionalcommits.org/
- **GitHub Actions:** https://docs.github.com/en/actions
