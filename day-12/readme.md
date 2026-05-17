# Day 12: Databases, Software Lifecycle, and Why Docker Exists

## What I Learned Today

### Types of Databases

#### SQL (Relational Databases)
- Structured data in tables with rows and columns
- Relationships between tables (foreign keys)
- Schema must be defined before adding data
- ACID compliant (Atomicity, Consistency, Isolation, Durability)
- Uses SQL (Structured Query Language) for queries

**Examples:**
- MySQL
- PostgreSQL
- Oracle
- Microsoft SQL Server
- SQLite



#### NoSQL (Non-Relational Databases)
- Flexible schema (or no schema)
- Can store unstructured or semi-structured data
- Designed for horizontal scaling
- Eventual consistency (not always ACID)

**Types of NoSQL Databases:**

1. **Document Databases**
   - Store data as JSON-like documents
   - Examples: MongoDB, CouchDB
   - Use case: Content management, user profiles, catalogs

2. **Key-Value Stores**
   - Simple key-value pairs
   - Examples: Redis, DynamoDB
   - Use case: Caching, session storage, real-time analytics



**When to Use NoSQL:**
- Rapidly changing data structures
- Massive scale (millions of users)
- Need horizontal scaling
- Real-time applications
- Big data and analytics



#### SQL vs NoSQL

| Feature | SQL | NoSQL |
|---------|-----|-------|
| **Schema** | Fixed, predefined | Flexible, dynamic |
| **Scaling** | Vertical (bigger servers) | Horizontal (more servers) |
| **Consistency** | ACID (strong) | Eventual (weaker) |
| **Relationships** | Built-in (JOINs) | Application handles it |
| **Use Case** | Structured data, transactions | Unstructured, big data, speed |
| **Examples** | Banking, e-commerce | Social media, IoT, analytics |

**Reality:** Most companies use both. SQL for core data, NoSQL for specific use cases.

### Software Lifecycle: From Code to Deployment

Today I finally understood the full journey software takes from developer's laptop to production.

#### The Lifecycle
Source Code → Dependencies → Build Tools → Artifacts → Deployment → Running Application

#### 1. Source Code
- What developers write
- Languages: Python, Java, JavaScript, Go, etc.
- Lives in version control (Git)

#### 2. Dependencies
- External libraries and packages your code needs
- Example: If you're building a web app, you need a web framework
- **Problem:** Different versions, conflicts, missing packages

**Package Managers handle this:**
- **Python:** pip, conda
- **JavaScript/Node:** npm, yarn
- **Java:** Maven, Gradle
- **Linux:** apt, yum, dnf
- **Ruby:** gem

Package managers download and install dependencies automatically.

#### 3. Build Tools
- Compile code (if needed)
- Run tests
- Package everything together
- Create deployable artifacts

**Examples:**
- **Java:** Maven, Gradle
- **JavaScript:** webpack, Rollup, Vite
- **Go:** go build
- **Python:** setuptools, poetry
- **C/C++:** make, CMake

#### 4. Artifacts
- The packaged, ready-to-deploy version of your application
- **Examples:**
  - Java: `.jar` or `.war` files
  - Python: `.whl` (wheel) files
  - JavaScript: bundled `.js` files
  - Go: compiled binary
  - **Docker: container images** (more on this below)

#### 5. Deployment
- Taking the artifact and running it in production
- **Traditional deployment problems:**
  - "Works on my machine" syndrome
  - Different OS versions on servers
  - Missing system libraries
  - Dependency conflicts
  - Configuration drift between environments

This is where Docker changed everything.

### Why Docker Exists (And Why It's So Popular)

#### The Problem Docker Solved

Before Docker, deployment looked like this:

1. Developer writes code on macOS
2. Code works perfectly on their laptop
3. Push to Git
4. Try to deploy on Ubuntu server
5. **Everything breaks**
   - Different Python version
   - Missing system libraries
   - Different file paths
   - Environment variables not set
   - Dependencies conflict with other apps on same server

"Works on my machine" wasn't a joke. It was a real nightmare costing companies millions in debugging time.

#### The Docker Solution

Docker packages **everything** your application needs to run into a single container image:
- Your application code
- All dependencies (libraries, packages)
- The exact runtime (Python 3.11, Node 18, etc.)
- System libraries
- Configuration files
- Even parts of the operating system

**Docker Image** = your application + everything it needs to run, frozen in time.

#### Why Docker Images Are Universal Deployable Artifacts
Traditional Artifact (e.g., .jar file):

Just your compiled code
Assumes the server has Java installed
Assumes correct version of Java
Assumes all system libraries present
Breaks if any assumption is wrong

Docker Image:

Your code + exact Java version + all libraries + OS dependencies
Run it on any machine with Docker installed
Same behavior everywhere: dev laptop, staging server, production
No "works on my machine" problem


#### The Docker Guarantee

If it runs in a Docker container on your laptop, it will run identically:
- On your teammate's Windows machine
- On the staging server (Ubuntu)
- On production (Amazon Linux)
- On your manager's MacBook

**Why?** Because the container IS the environment. You're not deploying to different environments, you're deploying the same environment everywhere.

#### How Docker Fits in the Software Lifecycle
Old Way:
Source Code → Build → Artifact (.jar) → Deploy to Server (hope it works)
Docker Way:
Source Code → Build → Docker Image (artifact) → Run Container Anywhere

The Docker image became the new universal artifact format because it's completely self-contained.



Docker containers share the host OS kernel but are isolated from each other. Much lighter than VMs.



## Key Takeaways
1. SQL for structured data and relationships, NoSQL for flexibility and scale
2. Software goes through a lifecycle: code → dependencies → build → artifact → deploy
3. Package managers handle dependencies, build tools create artifacts
4. Docker solved "works on my machine" by packaging environment + code together
5. Docker images are universal artifacts because they contain everything needed to run
6. Understanding *why* Docker exists matters more than just knowing Docker commands

## Why Today Was Mostly Theory
Can't use Docker effectively without understanding the problem it solves. Today was about seeing the full picture—how software moves from code to production, and why Docker became essential in that process.


**Progress: 12/90 days complete**