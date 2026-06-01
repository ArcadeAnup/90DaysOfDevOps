# Day 27: GitLab Exploration and Three-Tier Architecture Project

## GitLab vs GitHub (From a DevOps Perspective)

### GitHub
- Built for developers
- CI/CD through Actions (external tool)
- Great community
- More social features

### GitLab
- Built with DevOps in mind
- CI/CD pipelines built in
- Container registry built in
- Security scanning built in
- Can self-host (on-premise option)

**Quick Verdict:** GitHub for developers, GitLab for DevOps teams.

## Created New GitLab Repository

- Set up account on gitlab.com
- Explored project structure
- Created new project for three-tier app
- Explored built-in features (CI/CD, registries, wiki)

## Three-Tier Architecture Deep Dive

### Why Three-Tier?
User → Frontend (React/Vue) → Backend API → Database

Most real applications follow this pattern:
- **Tier 1 (Presentation):** User interface
- **Tier 2 (Application):** Business logic
- **Tier 3 (Data):** Persistent storage

### Why This Matters for DevOps

Each tier:
- Scales independently
- Deploys independently
- Has different resource requirements
- Can be containerized separately

Example:
High user load → Scale frontend containers
Heavy computation → Scale backend containers
More data → Add database replicas

Not just "deploy one thing." Deploy three things that work together.

## Resource Used

Video Series: https://www.youtube.com/watch?v=IUpsu2xemrA&list=PLdpzxOOAlwvIKMhk8WhzN1pYoJ1YU8Csa&index=40

This series walks through building a real three-tier application from scratch.


## The Learning Shift

There's a massive gap between:
- "I can run a Docker container"
- "I can containerize a three-tier application and deploy it"

First one is tool knowledge. Second one is system understanding.


## Key Insight

For 26 days I was collecting tools. Today I started building systems.

That's the difference between learning DevOps and actually doing DevOps.

---

**Progress: 27/90 days complete**

**Momentum:** This is the inflection point. From here on, everything connects.