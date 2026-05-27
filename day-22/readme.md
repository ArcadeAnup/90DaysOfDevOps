# Day 22: Comprehensive Revision and Consolidation

## Why Revision Matters

Three weeks in, I've covered:
- Linux fundamentals and networking
- Git and GitHub workflows
- AWS basics and CLI
- Docker, Docker Compose, networking
- Shell scripting
- Configuration management intro

Moving fast feels productive, but there's a difference between "I've seen this" and "I can use this without Googling."

Today was consolidation day. No new concepts. Just drilling what I already know until it becomes muscle memory.

## Linux Networking Commands

### Basic Network Inspection

**ip link**
- Shows all network interfaces (eth0, wlan0, lo, etc.)
- Use when: Checking if network card/interface is UP or DOWN

**ip addr**
- Shows IP addresses assigned to interfaces
- Use when: Checking your machine's IP, debugging internet/network issues

**ip route**
- Shows routing table and where packets go
- Use when: Internet works on one machine but not another, debugging gateway problems

**IPv4 vs IPv6**
- IPv4: 32 bits, 4.3 billion unique addresses
- IPv6: 128 bits, 3.8×10^38 unique addresses
- Check IPv4: `ip -4 addr`
- Check IPv6: `ip -6 addr`

### Connectivity Testing

**ping google.com**
- Tests if server/device is reachable
- Use when: Testing internet connectivity

**curl google.com**
- Sends HTTP request to websites/APIs
- Use when: Testing APIs, checking if website/server responds

**dig google.com**
- DNS lookup tool
- Use when: DNS not resolving, checking domain IPs

### Network Analysis

**netstat**
- Shows details about your network system
- `netstat -at` - only TCP connections
- `netstat -l` - all active ports
- `netstat -u` - all UDP connections

**ss -tulpn**
- Modern replacement for netstat
- Shows all listening ports and services

**tcpdump**
- Deep packet inspection and traffic analysis
- Use when: Need to see actual packets being transmitted

**traceroute google.com**
- Shows the path packets take to reach destination
- Use when: Debugging routing issues

### Remote Access

**SSH into Cloud Instance**
```bash
ssh -i <public-key> <username>@<ip-address>
```

## Linux System Information

### uname - System Information
```bash
uname              # Prints kernel/system info
uname -s           # Kernel name
uname -v           # Kernel version
uname -p           # Processor architecture
uname -a           # Shows everything together
```

### uptime - System Stability
```bash
uptime             # Current time, uptime, logged users, load averages
uptime -p          # Pretty human-readable uptime
uptime -s          # Exact boot time of system
```

### Distribution Information
```bash
cat /etc/os-release   # Linux distribution details
cat /etc/issue        # OS issue information
```

### Process Monitoring
```bash
w                  # Shows all running processes and logged-in users
lsof -u username   # Shows files opened by specific user
```

## Linux Fundamentals

### User and Group Management
```bash
whoami                              # Current user
su                                  # Switch to root user
sudo bash                           # Switch to root user with sudo
sudo useradd demo                   # Create user
sudo passwd demo                    # Set password for user
sudo userdel demo                   # Delete user
sudo groupadd techies               # Create group
sudo groupdel techies               # Delete group
```

### File Operations
```bash
touch filename                      # Create empty file
cat filename                        # Display file content
cp source destination               # Copy file
mv source destination               # Move/rename file
rm filename                         # Remove file
mkdir dirname                       # Create directory
rmdir dirname                       # Remove empty directory
```

### File Permissions and Ownership
```bash
chmod 755 file                      # Change file permissions
chown user:group file               # Change file ownership
```

### Search and Filter
```bash
grep "text" filename                # Search text in file
grep "^d" filename                  # Lines starting with 'd'
grep "d$" filename                  # Lines ending with 'd'
sort filename                       # Sort file content
ls | grep "pattern"                 # Pipe with grep
```

### Other Useful Commands
```bash
man command                         # Manual page for command
clear                               # Clear terminal
pwd                                 # Print working directory
cd /path                            # Change directory
echo "text"                         # Print text
```

## Pipes and Redirection

**Pipe (|)**
- Sends output of one command as input to another
- Example: `ls | grep "txt"` - list files, filter for .txt
- Very important in Linux scripting and DevOps

**Redirection**
- `>` - Redirect output to file (overwrite)
- `>>` - Append to file
- `<` - Input from file

## Docker Troubleshooting Checklist

### Container Issues

**Check Running Containers**
```bash
docker ps
```

**Check All Containers (including stopped)**
```bash
docker ps -a
```

**Check Available Images**
```bash
docker images
```

**View Container Logs**
```bash
docker logs container_ID          # View logs
docker logs -f container_ID       # Stream live logs
```

### Container Operations

**Run Container**
```bash
docker run image-name             # Foreground
docker run -d image-name          # Background (detached)
```

**Access Container**
```bash
docker exec -it container_ID /bin/bash    # Open shell
```

**Container Lifecycle**
```bash
docker stop container_ID          # Stop container
docker start container_ID         # Start container
docker restart container_ID       # Restart container
docker rm container_ID            # Remove container
```

### Port and Network

**Port Mapping**
```bash
docker run -p host_port:container_port image-name
```

**Check Docker Networks**
```bash
docker network ls                 # List all networks
docker network create network-name # Create network
docker network inspect network-name # View network details
```

**Connect Container to Network**
```bash
docker network connect network-name container_ID
```

## Troubleshooting Workflows

### Internet/Networking Not Working

Do this in order:

1. **Check IP**
```bash
   ip addr show
```

2. **Check Interface Status**
```bash
   ip link
```

3. **Ping Test**
```bash
   ping google.com
```

4. **DNS Check**
```bash
   dig google.com
```

5. **Routing Check**
```bash
   ip route
```

6. **Port Check**
```bash
   ss -tulpn
```

7. **Traffic Analysis**
```bash
   tcpdump
```

8. **Restart Network Services**
```bash
   sudo systemctl restart NetworkManager
```

9. **Deep Packet Debugging**
```bash
   traceroute google.com
```

### Docker Container Not Working

Do this in order:

1. Check if container is running: `docker ps`
2. Check container logs: `docker logs container_ID`
3. Enter container: `docker exec -it container_ID /bin/bash`
4. Check port mapping: `docker run -p host_port:container_port`
5. Check networks: `docker network ls`
6. Verify container is on correct network: `docker network inspect network-name`
7. Test connectivity from container: `docker exec container_ID ping other-container`



**Progress: 22/90 days complete**

**Note:** This document consolidates cheat sheets, common troubleshooting patterns, and fundamental commands from Days 1-21. It's a working reference, not polished documentation. Use it as a quick lookup when you need to remember something fast.

BTW CARPEDIAM DOSTO.........