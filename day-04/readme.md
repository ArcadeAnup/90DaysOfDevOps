# Day 4: AWS EC2, Linux Commands, and Server Basics

## What I Learned Today

### AWS EC2 Instance
- Created my first AWS account
- Launched an EC2 instance (Elastic Compute Cloud)
- Watched it go from "pending" to "running" (weirdly satisfying)
- EC2 is basically renting a computer in the cloud that you can SSH into and configure however you want

### Client OS vs Server OS
- **Client OS**: Designed for end users. Has a GUI, optimized for desktop applications, user-friendly
- **Server OS**: Designed to run services and handle requests. No GUI (usually), optimized for stability and performance, runs 24/7
- Examples: Windows 10/11 vs Windows Server, Ubuntu Desktop vs Ubuntu Server
- Server OS is lighter because it doesn't need all the visual stuff eating resources

### Linux Commands (Hands-On Practice)
Spent most of the day running commands until they stuck:

#### File and Directory Operations
- `ls` - list files
- `cd` - change directory
- `mkdir` - make directory
- `rm` - remove files
- `cp` - copy files
- `mv` - move/rename files
- `touch` - create empty file
- `cat` - display file contents

#### File Permissions
- `chmod` - change file permissions
- `chown` - change file owner
- Permission structure: read (r), write (w), execute (x)
- Format: `rwxrwxrwx` (owner, group, others)
- Example: `chmod 755 file.sh` gives owner full permissions, group and others read+execute

#### User and Group Management
- `useradd` - add new user
- `usermod` - modify user
- `groupadd` - add new group
- `su` - switch user
- `sudo` - run command as superuser

### Linux Users and Groups
- **Users**: Individual accounts on the system
- **Groups**: Collection of users with shared permissions
- Every file has an owner and a group
- Permissions apply to: owner, group, everyone else
- Root user = superuser with all permissions (use carefully)

### The Linux Kernel
- The kernel is the core of the OS
- Sits between hardware and applications
- Manages: memory, processes, hardware drivers, system calls
- You interact with the kernel through the shell (bash, zsh, etc.)
- Think of it as the translator between "I want to open this file" and the actual hardware operations needed

### NGINX Introduction
- NGINX is a web server (and reverse proxy, load balancer, etc.)
- Powers a massive chunk of the internet
- Lightweight and fast compared to Apache
- Handles multiple requests simultaneously without breaking a sweat
- Common use case: serving static files, routing traffic to backend servers

## Key Takeaways
1. EC2 instances are just virtual machines you control remotely
2. Linux permissions are a bouncer system for your files
3. The kernel does all the real work, you just tell it what to do
4. Server OS strips out the GUI and focuses on running services
5. NGINX is everywhere and I should probably learn it properly




**Progress: 4/90 days complete**