# Day 8: Git, Feature Branches, and Realizing I've Been Doing This Wrong

## What I Learned Today

### Git vs GitHub
- **Git**: Version control system that tracks changes in your code
- **GitHub**: Platform that hosts Git repositories online and adds collaboration features
- Git works locally on your machine, GitHub is where you push it for backup/sharing

### Git Basics I Already Knew
- `git init` - initialize a repository
- `git add` - stage files for commit
- `git commit -m "message"` - save changes with a description
- `git push` - send commits to GitHub
- `git pull` - get latest changes from GitHub
- `git status` - check what's changed
- `git log` - see commit history

### What I Actually Learned Today: Branching

#### Feature Branches
- Branches let you work on new features without touching the main codebase
- Create a branch: `git branch feature-name`
- Switch to it: `git checkout feature-name`
- Or do both at once: `git checkout -b feature-name`
- Work on your feature, commit changes, then merge back to main when ready
- If something breaks, main is still safe

#### Hotfix Branches
- For critical bugs that need immediate fixes
- Create from main: `git checkout -b hotfix-bug-name`
- Fix the bug, test it, merge back quickly
- Doesn't disrupt ongoing feature development
- Can deploy the fix without waiting for features to finish

#### Main Branch
- Should always be stable and deployable
- Don't commit directly to main (I've been doing this like an idiot)
- Only merge tested, working code into main
- Think of it as the "production-ready" version

### Common Git Workflow

```bash
# Start working on a new feature
git checkout -b feature-new-script

# Make changes, stage them
git add script.sh
git commit -m "Add AWS monitoring script"

# Push feature branch to GitHub
git push origin feature-new-script

# When feature is done and tested, merge to main
git checkout main
git merge feature-new-script

# Push updated main
git push origin main

# Delete the feature branch (optional)
git branch -d feature-new-script
```

### Other Useful Git Commands

```bash
git branch                    # List all branches
git branch -d branch-name     # Delete a branch
git merge branch-name         # Merge branch into current branch
git diff                      # See unstaged changes
git diff --staged             # See staged changes
git reset HEAD file           # Unstage a file
git checkout -- file          # Discard changes in file
git clone url                 # Clone a repository
```

## What I Realized Today
I've been committing directly to main since Day 1. Which technically works when you're solo and just pushing daily notes. But it's terrible practice for actual development work.

The moment you break something, you have no clean rollback point. No way to experiment safely. No separation between "work in progress" and "ready to deploy."

Feature branches solve this. You can try things, break stuff, commit messy code—all without touching the stable version. If it works, merge it. If it doesn't, delete the branch and pretend it never happened.

## Key Takeaways
1. Git commands are easy to learn, Git workflow is what actually matters
2. Never commit directly to main in real projects
3. Feature branches = experiment safely
4. Hotfix branches = fix critical bugs without disrupting development
5. I've been using version control wrong for a week and nobody told me

## Mistakes I Made
- Been committing everything directly to main
- Didn't understand why branching existed


**Progress: 8/90 days complete**