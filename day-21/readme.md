```markdown
# Day 21: Docker Compose and Three-Tier Architecture

## What I Learned

### Docker Compose Basics
- YAML file defines entire multi-container application
- One command to start everything: `docker-compose up`
- One command to stop everything: `docker-compose down`
- Automatic networking between services
- Environment variables, ports, volumes all in one place

### Why Docker Compose Matters
Before Docker Compose (painful):
```bash
docker network create app-network
docker run -d --name db --network app-network -v db-data:/data postgres
docker run -d --name api --network app-network -e DB_HOST=db backend-image
docker run -d --name frontend --network app-network -p 3000:3000 frontend-image
# And if you need to stop it, reverse all those commands
```

With Docker Compose (simple):
```bash
docker-compose up
# Everything runs
docker-compose down
# Everything stops
```

### Docker Compose File Structure
```yaml
version: '3.8'

services:
  frontend:
    image: frontend-image
    ports:
      - "3000:3000"
    networks:
      - app-network

  api:
    image: backend-image
    environment:
      - DB_HOST=db
    networks:
      - app-network
    depends_on:
      - db

  db:
    image: postgres
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

## Three-Tier Architecture Demo

Studied the three-tier architecture repository: https://github.com/iam-veeramalla/three-tier-architecture-demo/tree/master

### What is Three-Tier Architecture?

**Tier 1 (Presentation):** Frontend UI
- React/Vue/Angular app
- User-facing interface
- Port 3000 typically

**Tier 2 (Application):** Backend API
- Express, Django, Spring, etc.
- Business logic
- Port 5000 or 8000 typically

**Tier 3 (Data):** Database
- PostgreSQL, MySQL, MongoDB
- Persistent data storage
- Port 5432 (PostgreSQL) internally only

### How They Communicate

```
User Browser
    ↓
Frontend Container (port 3000)
    ↓
API Container (talks to frontend)
    ↓
Database Container (talks to API)
```

All on the same Docker network, so they can reach each other by service name.

### The Docker Compose Setup

```yaml
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - api

  api:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - DB_HOST=db
      - DB_USER=postgres
      - DB_PASSWORD=secret
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Key Features

**depends_on:**
```yaml
depends_on:
  - db
```
Ensures database starts before API (prevents connection errors)

**Environment Variables:**
```yaml
environment:
  - DB_HOST=db
  - DB_PASSWORD=secret
```
API knows to reach database at hostname `db` (DNS resolution via Docker network)

**Volumes:**
```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```
Database data persists even when containers stop

**Ports:**
```yaml
ports:
  - "3000:3000"  # Only frontend exposed to host
```
Only frontend needs external access. API and DB are internal only.

## Commands I Used

```bash
# Start all services
docker-compose up

# Start in background
docker-compose up -d

# View logs
docker-compose logs

# Logs for specific service
docker-compose logs api

# Stop all services
docker-compose down

# Remove volumes too
docker-compose down -v

# List running services
docker-compose ps

# Execute command in service
docker-compose exec api bash

# Rebuild images
docker-compose build
```

## What Made It Click

Running a three-tier app with one command showed me:
- How microservices actually work in practice
- How networking connects services automatically
- Why environment variables matter (so services know where to find each other)
- How volumes keep data safe
- The entire production-like architecture running locally

This is what Docker Compose was built for. Not single containers. Multi-container applications.

## Real-World Impact

Before Docker Compose:
- Setup took 20+ manual commands
- Easy to make mistakes
- Startup order mattered (manual coordination)
- Hard to share with team (how do I explain all those commands?)

With Docker Compose:
- One command: `docker-compose up`
- Single YAML file = complete documentation
- Anyone with Docker can run the entire app
- Same setup locally as production (with differences only in env vars)

## Key Takeaways
1. Docker Compose eliminates manual `docker run` commands
2. YAML file documents entire architecture
3. Automatic networking between services
4. `depends_on` ensures startup order
5. Environment variables let services find each other
6. Volumes keep data persistent across restarts
7. Three-tier = frontend, backend, database (most common architecture)

## What I Built
- Cloned three-tier architecture demo repo
- Ran `docker-compose up`
- All three services started automatically
- Tested connectivity between frontend → API → database
- Explored logs and service interaction
- Stopped and restarted with `docker-compose down/up`

**Progress: 21/90 days complete**
```
