# Day 11: Revision, YAML Introduction, and Configuration Management (Ansible vs Puppet)

## What I Did Today

### Revision: Day 1-10
Went back through all my notes from the past 10 days. This is a marathon, not a sprint

#### What Clicked During Revision
- File permissions finally make complete sense now
- Git branching workflow is muscle memory now after reviewing and practicing
- AWS services connections are clearer—see them as an ecosystem, not isolated tools
- Shell scripting patterns are starting to feel natural instead of intimidating

**Key Insight**: Reading something once doesn't mean you learned it. Coming back to concepts makes them stick.

## What I Learned Today

### YAML (YAML Ain't Markup Language)
- Human-readable data serialization language
- Used EVERYWHERE in DevOps: Docker, Kubernetes, Ansible, CI/CD pipelines
- Similar to JSON but more readable
- Syntax is whitespace-sensitive (like Python)

#### YAML Basics

```yaml
# Comments start with #
# Key-value pairs
name: Shubh
age: 21
learning: ML

# Lists
skills:
  - Linux
  - Git
  - AWS
  - Shell Scripting

# Nested structures
person:
  name: Shubh
  role: Student
  interests:
    - DevOps
    - Cloud Computing
    - Automation

# Inline list
tools: [Docker, Kubernetes, Ansible]

# Multi-line strings
description: |
  This is a multi-line
  string that preserves
  line breaks.

# Boolean values
is_learning: true
gave_up: false

# Null values
middle_name: null
```


#### Why YAML Matters in DevOps
- Docker Compose files: YAML
- Kubernetes manifests: YAML
- Ansible playbooks: YAML
- GitLab CI/CD pipelines: YAML
- GitHub Actions workflows: YAML

If you're doing DevOps, you're writing YAML. A lot of it.

### Configuration Management

**Configuration Management** = automating the setup, configuration, and maintenance of servers and infrastructure.

Instead of manually SSH-ing into every server to:
- Install packages
- Update configurations
- Deploy applications
- Manage users and permissions

You write code that does it automatically across hundreds or thousands of machines.

#### Why Configuration Management Matters
- **Consistency**: Every server configured the same way
- **Speed**: Configure 100 servers as fast as 1 server
- **Documentation**: Your infrastructure is defined in code
- **Disaster Recovery**: Rebuild everything from code if needed
- **Version Control**: Track changes to infrastructure like you track code

### Ansible

#### What is Ansible?
- Open-source configuration management and automation tool
- Created by Red Hat
- Uses YAML for configuration (playbooks)
- **Agentless**: No software to install on managed machines
- Uses SSH to connect to servers
- Push-based model: you push configurations from control node

#### Key Concepts
- **Control Node**: Machine where Ansible is installed (your laptop)
- **Managed Nodes**: Servers you're configuring
- **Inventory**: List of servers to manage
- **Playbooks**: YAML files that define what to do
- **Modules**: Pre-built functions (install package, copy file, etc.)
- **Tasks**: Individual actions in a playbook
- **Roles**: Reusable collections of tasks



#### Ansible Pros
- Easy to learn (YAML + SSH knowledge = you're good)
- No agents to manage
- Simple setup
- Large community and modules



#### Puppet Pros
- Powerful for large, complex infrastructures
- Pull-based model = agents are independent
- Strong reporting and compliance features
- Mature ecosystem
- Good for enterprises

#### Puppet Cons
- Steeper learning curve (custom DSL)
- Requires agent installation and management
- More complex setup
- Overkill for small projects

### Ansible vs Puppet

| Feature | Ansible | Puppet |
|---------|---------|--------|
| **Architecture** | Agentless (SSH) | Agent-based (master-agent) |
| **Language** | YAML | Puppet DSL |
| **Model** | Push (you initiate) | Pull (agents check in) |
| **Learning Curve** | Easy | Steeper |
| **Setup** | Simple | More complex |
| **Best For** | Small to medium infra | Large, complex infra |
| **Performance** | Can be slower at scale | Better for huge scale |
| **Use Case** | Quick automation, cloud | Enterprise, compliance |



## Key Takeaways
1. Revision is not wasted time—it's where learning actually sticks
2. YAML is unavoidable in DevOps, better get comfortable with it
3. Configuration Management = automating server setup and maintenance
4. Ansible is agentless and easy to learn (start here)
5.YAML is Easy to Learn as its a Markup Language but you need to be either a fbi agent or pro geoguessrplayer to debug it.

## Why Today Was Theory-Heavy
Sometimes you need conceptual understanding before jumping into hands-on. Configuration Management is a big topic—understanding what problem it solves and how different tools approach it matters before writing playbooks blindly.

Tomorrow: hands-on Ansible.

**Progress: 11/90 days complete**