# Day 10: GitLab, Branching Strategies, AWS Services Introduction, and Git Practice

## What I Learned Today

### GitLab vs GitHub

#### GitHub
- Most popular Git hosting platform
- Great for open-source projects
- Strong community and social features
- CI/CD through GitHub Actions (separate setup)
- Owned by Microsoft

#### GitLab
- Built specifically with DevOps in mind
- CI/CD pipelines integrated out of the box
- Built-in container registry
- Security scanning and vulnerability detection
- Auto DevOps features
- Can self-host (on-premise option)
- Everything in one platform

#### Key Differences
- **GitLab**: All-in-one DevOps platform, better for enterprise and CI/CD-heavy workflows
- **GitHub**: Better community, more third-party integrations, simpler for basic version control
- GitLab feels like it was built for DevOps teams, GitHub feels like it was built for developers

### Created GitLab Repository
- Made an account on GitLab
- Created a new repository
- Explored the interface (way more DevOps-focused than GitHub)
- Noticed built-in CI/CD pipeline options right from the start
- Container registry is just... there. No setup needed.

### Branching Strategies

#### Trunk-Based Development
- Everyone works on the main branch (trunk)
- Very short-lived feature branches (hours or a day max)
- Frequent small commits directly to main
- Requires strong test coverage and CI/CD
- Fast deployment cycles
- Used by: Google, Facebook, high-velocity teams

**Pros:**
- Fast integration
- Less merge conflict hell
- Simpler workflow
- Continuous delivery friendly

**Cons:**
- Requires discipline and good tests
- Can be chaotic without proper CI/CD
- Risky if team isn't experienced

#### Feature Branch Strategy
- Each feature gets its own branch
- Work isolated from main until feature is complete
- Merge to main when ready and tested
- More common in smaller teams or open-source
- What I've been doing so far

**Pros:**
- Safe experimentation
- Easy to review before merging
- Main stays stable
- Good for teams learning Git

**Cons:**
- Longer-lived branches = more merge conflicts
- Slower integration
- Can delay feedback

#### Git Flow (Another Strategy)
- More complex branching model
- Branches: main, develop, feature, release, hotfix
- Structured release cycles
- Good for scheduled releases
- Overkill for small projects

#### When to Use What
- **Trunk-based**: Fast-moving teams, strong CI/CD, experienced developers
- **Feature branches**: Small to medium teams, learning Git, need code review
- **Git Flow**: Large teams, scheduled releases, complex projects

The strategy depends on team size, deployment frequency, and how mature your DevOps practices are. Not just personal preference.

### Introduction to AWS Services

#### EC2 (Elastic Compute Cloud)
- Virtual servers in the cloud
- You rent computing power
- Launch instances, install whatever you want
- What I've been using for Linux practice

#### S3 (Simple Storage Service)
- Object storage service
- Store files, images, backups, anything
- Pay for what you use
- Super reliable (99.999999999% durability)
- Used by basically everyone

#### Lambda
- Serverless compute service
- Run code without managing servers
- Pay only when code executes
- Event-driven (trigger on file upload, API call, etc.)
- Great for small tasks and automation

#### RDS (Relational Database Service)
- Managed database service
- Supports MySQL, PostgreSQL, Oracle, SQL Server
- Automatic backups, updates, scaling
- Don't have to manage database infrastructure yourself

#### How They Fit Together (Example)
- User uploads file → S3
- S3 triggers Lambda function
- Lambda processes file, stores data in RDS
- EC2 instance serves web app that queries RDS
- All connected, all automated

Starting to see AWS as a connected ecosystem instead of random services.

### Git Quizzes
Tested myself on:
- Branching and merging
- SSH vs HTTPS authentication
- Git workflow commands
- Resolving basic scenarios (when to use what command)
- Forking vs cloning

Retained way more than I expected. The hands-on practice from Day 9 made concepts stick.

## Key Takeaways
1. GitLab is genuinely more DevOps-friendly than GitHub, not just marketing
2. Branching strategies are real methodologies with tradeoffs, not random choices
3. Trunk-based development requires discipline, feature branches are safer for learning
4. AWS services are designed to work together, not exist in isolation
5. Testing yourself reveals what you actually learned vs what you just read


**Progress: 10/90 days complete**