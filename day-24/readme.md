# Day 24: Jenkins Master-Agent Architecture and First Real CI Pipeline

## What I Learned

### Jenkins Master-Agent Architecture
- **Master**: Orchestrates builds, manages jobs, UI
- **Agent**: Runs actual builds (can be multiple machines)
- Master distributes work across agents for parallel execution
- Scalable: add more agents to handle more builds

**Local Setup Problem:** VirtualBox resource-heavy, overkill for learning

**Solution:** Use cloud-native CI tools (GitHub Actions, GitLab CI)

## Built First Real CI Pipeline: GitHub Actions

**Trigger:** Push code to GitHub → Workflow automatically runs

**What the Workflow Does:**
- Install Node.js
- Install dependencies
- Run tests
- Report results in PR

## GitHub Actions Workflow (YAML)

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
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

## Matrix Builds

```yaml
strategy:
  matrix:
    node-version: [18.x, 20.x, 22.x]
```

**What This Does:**
- Run tests on Node 18, 20, 22 in parallel
- Same code, multiple environments
- All pass/fail independently
- See results for all versions at once

**Result:** ✅ ✅ ✅ All green

## Why GitHub Actions > Jenkins (for Learning)

| | Jenkins | GitHub Actions |
|---|---------|---|
| **Setup** | Install, configure, agents | Use what's built in |
| **Infrastructure** | Manage VMs/servers | Cloud-hosted |
| **Complexity** | Master-Agent architecture | Just YAML file |
| **Resource Usage** | Heavy (VirtualBox) | Free (GitHub provides) |
| **Integration** | Needs plugins | Native GitHub integration |

For learning: GitHub Actions wins.

## CI/CD Workflow
Code Push → GitHub detects change →
Workflow triggers → Build & test →
Results in PR → Merge if green

No manual intervention. Automatic.

## Key Takeaways
1. Master-Agent is scalable architecture (Jenkins)
2. Cloud-native CI tools better for learning (GitHub Actions)
3. Matrix builds = test multiple environments in parallel
4. CI/CD is automatic testing on code changes
5. Green checks ✅ = build succeeded



**Progress: 24/90 days complete**