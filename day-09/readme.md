# Day 9: Git Hands-On - SSH, PAT, Forking, and Real Practice

## What I Learned Today

### Forking Repositories
- **Fork** = creating your own copy of someone else's repository
- Lets you experiment with changes without affecting the original project
- Common workflow for contributing to open-source projects
- Fork button on GitHub → creates a copy under your account
- You can make changes, then submit a pull request to the original repo

### GitHub Access Methods

#### HTTPS vs SSH
- **HTTPS**: Uses username/password or Personal Access Token
  - Simple to set up
  - Works everywhere
  - Have to authenticate every time (annoying)
- **SSH**: Uses cryptographic key pairs
  - More secure
  - No password prompts once set up
  - Industry standard for DevOps work

### Setting Up SSH Keys

#### Generate SSH Key
```bash
# Generate new SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Or if ed25519 isn't supported
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Press Enter to accept default location (~/.ssh/id_ed25519)
# Set a passphrase (optional but recommended)
```

#### Add SSH Key to SSH Agent
```bash
# Start the SSH agent
eval "$(ssh-agent -s)"

# Add your SSH private key
ssh-add ~/.ssh/id_ed25519
```

#### Add Public Key to GitHub
```bash
# Copy your public key
cat ~/.ssh/id_ed25519.pub

# Go to GitHub → Settings → SSH and GPG keys → New SSH key
# Paste the public key and save
```

#### Test SSH Connection
```bash
ssh -T git@github.com
# Should see: "Hi username! You've successfully authenticated..."
```

### Personal Access Tokens (PAT)
- Alternative to passwords for HTTPS authentication
- More secure than regular passwords
- Can set specific permissions (repo access, read/write, etc.)
- Can be revoked without changing your GitHub password

#### Creating a PAT
1. GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Generate new token
3. Select scopes (permissions)
4. Copy the token (you won't see it again)
5. Use it as your password when Git asks for credentials

### Cloning Through SSH

#### SSH Clone
```bash
# Clone using SSH
git clone git@github.com:username/repo.git

# No authentication prompts if SSH is set up correctly
```

#### HTTPS Clone (with PAT)
```bash
# Clone using HTTPS
git clone https://github.com/username/repo.git

# When prompted for password, use your PAT
```

### Git Workflow Practice Today

```bash
# Fork a repository on GitHub (click Fork button)

# Clone your fork using SSH
git clone git@github.com:myusername/forked-repo.git
cd forked-repo

# Create feature branch
git checkout -b feature-new-functionality

# Make changes to files
# ... edit files ...

# Stage and commit
git add .
git commit -m "Add new functionality"

# Push to your fork
git push origin feature-new-functionality

# On GitHub, create Pull Request to original repo (if contributing)

# Switch back to main
git checkout main

# Pull latest changes
git pull origin main

# Merge feature branch locally
git merge feature-new-functionality

# Push updated main
git push origin main

# Delete feature branch
git branch -d feature-new-functionality
```

### Git Commands Practiced Today

```bash
git clone                     # Clone a repository
git fork                      # (Done through GitHub UI)
git remote -v                 # See remote URLs
git remote add upstream url   # Add original repo as upstream
git fetch upstream            # Get changes from original repo
git checkout -b branch        # Create and switch to branch
git add .                     # Stage all changes
git commit -m "message"       # Commit changes
git push origin branch        # Push branch to remote
git pull origin main          # Pull latest from main
git merge branch              # Merge branch into current
git branch -d branch          # Delete branch
```

## What I Actually Did Today
1. Forked an open-source repository
2. Generated SSH key pair on my machine
3. Added public key to GitHub
4. Cloned the forked repo using SSH (no password prompt)
5. Created multiple feature branches
6. Made changes, committed, pushed
7. Practiced merging branches
8. Deleted old branches after merging

No tutorials. Just repetition until the workflow felt natural.

## Key Takeaways
1. SSH authentication > HTTPS for regular Git work
2. Once SSH is set up, Git operations are smooth and passwordless
3. Forking is how you contribute to projects you don't own
4. PAT is more secure than passwords but SSH is industry standard
5. Practice matters more than reading documentation

## Why SSH Keys Work
- You have two keys: private (stays on your machine) and public (goes on GitHub)
- Private key proves you are who you say you are
- Public key lets GitHub verify the private key's signature
- No passwords transmitted over the network
- More secure and way more convenient

## Mistakes I Made
- Tried tdero clone with SSH before adding key to GitHub (obviously failed)
- Forgot to start SSH agent, woned why authentication wasn't working
- Generated key without email flag, had to redo it


**Progress: 9/90 days complete**