# Day 18: Docker Networking

## What I Learned

### Network Types
- **Bridge (default)**: Isolated network, containers get IPs but can't communicate by name
- **Custom Bridge**: Same as bridge but with DNS resolution (containers talk by name)
- **Host**: Container shares host network (no isolation)
- **None**: No network at all

### Default Bridge Problem
- Containers on default bridge can only communicate via IP addresses
- If container restarts, IP changes, everything breaks
- No DNS resolution

### Custom Bridge Solution
```bash
# Create network
docker network create mynetwork

# Run containers on it
docker run -d --name app1 --network mynetwork nginx
docker run -d --name app2 --network mynetwork alpine

# Now they can ping by name
docker exec app1 ping app2  # Works!
```

### Key Commands
```bash
docker network ls                           # List networks
docker network create                 # Create network
docker network inspect                # View details
docker run --network           # Run on network
docker network connect   # Attach to network
```

### Real-World Setup
```bash
# Create network
docker network create app-network

# Database (not exposed to host)
docker run -d --name db --network app-network postgres

# Web app (exposed to host, can reach db by name)
docker run -d --name web --network app-network -p 8000:8000 \
  -e DATABASE_URL=postgresql://db:5432/mydb my-app
```

## Key Takeaways
1. Always use custom bridge networks for multi-container apps
2. Custom bridge enables DNS (container names instead of IPs)
3. Network isolation = security (separate networks for frontend/backend/db)
4. Containers must be on same network to communicate

## What I Built
- Created custom bridge networks
- Ran containers on different networks
- Tested connectivity between containers
- Debugged subnet mismatch issues on default bridge
- Set up isolated networks for security

**Full detailed guide with diagrams:** [Docker Networking Made Easy (Blog Post)]

---

**Progress: 18/90 days complete**