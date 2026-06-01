# Day 26: GitHub Actions Self-Hosted Runner on Azure VM

## What is a Self-Hosted Runner?

**GitHub-Hosted Runner:**
- GitHub manages the server
- Limited customization
- Free quota (2000 minutes/month)
- Can't integrate with on-premise tools

**Self-Hosted Runner:**
- Your own hardware (Azure VM in this case)
- Full control and customization
- Unlimited execution time
- Can run Docker, Kubernetes, anything installed

## How It Works
Code Push → GitHub Actions triggered →
Connects to self-hosted runner (your Azure VM) →
Runs jobs on your VM →
Reports results back to GitHub

No intermediate servers. Direct connection to YOUR hardware.

## Setting Up Self-Hosted Runner

### 1. Create Runner in GitHub Repo

**Steps:**
1. GitHub Repo → Settings → Actions → Runners
2. Click "New self-hosted runner"
3. Select: Linux, X64



### 3. Run the Runner

```bash
# Start runner (listen for GitHub Actions jobs)
./run.sh
```

**Output:**
√ Connected to GitHub
√ Ready to accept jobs

Runner is now listening for GitHub Actions workflows.

## Update GitHub Actions Workflow

**Before (GitHub-hosted):**
```yaml
jobs:
  test:
    runs-on: ubuntu-latest    # GitHub's servers
```

**After (Self-hosted):**
```yaml
jobs:
  test:
    runs-on: self-hosted       # Your Azure VM
    strategy:
      matrix:
        node-version: [18.x, 20.x, 22.x]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
```

## Test the Runner

### 1. Trigger Workflow

```bash
# Push code to GitHub
git add .
git commit -m "Test self-hosted runner"
git push origin main
```

### 2. Watch Workflow Execute

**On GitHub UI:**
- Go to repo → Actions tab
- See workflow running
- Watch job progress in real-time

**On Azure VM Terminal:**
√ Running job: build
√ Step: Run npm ci
√ Step: Run tests
√ Job completed

Same job, visible in both places.

### 3. Verify Matrix Build Results

```yaml
matrix:
  node-version: [18.x, 20.x, 22.x]
```

**Result:**
- Node 18: ✅ PASSED
- Node 20: ✅ PASSED
- Node 22: ✅ PASSED

All three versions tested in parallel on your VM.

## What This Unlocks

### Before (GitHub-hosted runners)
Code Push → GitHub Actions (their servers) →
Results → Limited to what GitHub allows

### After (Self-hosted runner)
Code Push → GitHub Actions (triggers) →
Your Azure VM runs the job →
Full control: install anything, run anything

## Real-World Use Cases

**Enterprise:**
- Run builds on company's own infrastructure
- Integrate with on-premise tools (internal registries, private databases)
- Security: code never leaves company network
- Compliance: meet data residency requirements

**Open Source:**
- Run expensive GPU tests (self-hosted GPU machine)
- Integration tests with hardware (IoT projects)
- Tests requiring high resources (compile, large datasets)

**Local Development:**
- Test against real environment before deploying
- Run Docker, Kubernetes locally
- Test with actual databases/services



Service runs even if you disconnect from SSH.



## How This Differs From Day 25 Jenkins

**Day 25 (Jenkins on Azure):**
- Manual setup, manual job triggers
- Web UI interaction
- Self-contained CI system

**Day 26 (GitHub Actions self-hosted):**
- Automated setup from GitHub
- Triggered by code push automatically
- GitHub as orchestrator, your VM as executor
- Cloud CI + on-premise execution

## Key Concepts Learned

1. **Separation of concerns**: GitHub manages CI logic, your VM executes
2. **Scalability**: Can add multiple runners to distribute load
3. **Control**: Full access to VM environment and resources
4. **Security**: Code runs on your infrastructure
5. **Integration**: Can connect to private tools/databases


## Project Reference

GitHub Repo: https://github.com/ArcadeAnup/Learning_project

This repo has:
- Self-hosted runner configured
- Matrix build workflow (Node 18, 20, 22)
- Automated tests on every push
- Live execution on Azure VM


## What This Means

**Day 25:** "Jenkins is running publicly"
**Day 26:** "GitHub Actions is orchestrating CI on my infrastructure"

Different tools, different architecture, same goal: automated testing and deployment.

---

**Progress: 26/90 days complete**

**Milestone:** You're now running production-style CI/CD. GitHub as orchestrator, your Azure VM as executor. This is how real teams do it.