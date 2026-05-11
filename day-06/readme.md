# Day 6: SSH, Linux Networking Commands, and Shell Scripting Hands-On

## What I Learned Today

### SSH (Secure Shell)
- SSH lets you securely connect to remote servers over a network
- Basic syntax: `ssh username@ip_address`
- Example: `ssh ubuntu@52.12.34.56`
- Uses port 22 by default
- After connecting, you're basically controlling the remote machine through your terminal

#### SSH Key-Based Authentication
- More secure than password login
- Generate keys with `ssh-keygen`
- Creates two files: private key (keep secret) and public key (share with servers)
- Add public key to server's `~/.ssh/authorized_keys`
- Connect without typing password every time

#### Useful SSH Commands
- `ssh user@host` - connect to remote server
- `ssh -i keyfile.pem user@host` - connect using specific key file
- `exit` or `logout` - disconnect from server
- `scp file.txt user@host:/path/` - copy files to remote server
- `scp user@host:/path/file.txt .` - copy files from remote server

### Linux Networking Commands

#### `ping`
- Tests connectivity to another host
- Example: `ping google.com`
- Shows if packets are reaching the destination and how long it takes
- Use `Ctrl+C` to stop it

#### `ifconfig` / `ip`
- Displays network interface information
- Shows IP addresses, MAC addresses, network status
- `ifconfig` is older, `ip addr` is the modern replacement
- Example: `ip addr show`

#### `netstat`
- Shows network connections, routing tables, interface stats
- `netstat -tuln` - shows all listening ports
- Useful for seeing what services are running and on which ports

#### `curl`
- Transfer data from or to a server
- Example: `curl https://api.github.com`
- Can download files: `curl -O https://example.com/file.zip`
- Super useful for testing APIs and endpoints

#### `wget`
- Download files from the web
- Example: `wget https://example.com/file.zip`
- Can resume interrupted downloads
- Simpler than curl for basic downloads

#### `traceroute`
- Shows the path packets take to reach a destination
- Example: `traceroute google.com`
- Helps debug network routing issues

#### `nslookup` / `dig`
- DNS lookup tools
- Find IP address of a domain
- Example: `nslookup google.com`
- `dig` gives more detailed DNS information

### Shell Scripting Hands-On
Did more practical scripting today. Here's what I practiced:

#### Variables
```bash
#!/bin/bash
NAME="DevOps"
echo "Learning $NAME"
```

#### Conditionals
```bash
#!/bin/bash
if [ -f "file.txt" ]; then
    echo "File exists"
else
    echo "File does not exist"
fi
```

#### Loops
```bash
#!/bin/bash
for i in 1 2 3 4 5
do
    echo "Number: $i"
done
```

#### Reading User Input
```bash
#!/bin/bash
echo "Enter your name:"
read NAME
echo "Hello, $NAME"
```

## What I Built Today
1. A script that checks server connectivity using ping
2. A script that backs up files and logs the process
3. A network diagnostics script that runs multiple networking commands and saves output

All messy, all functional.

## Key Takeaways
1. SSH is how you actually work with remote servers in the real world
2. Networking commands are boring until you need to debug something, then they're essential
3. Shell scripting gets easier the more you just write scripts, even broken ones
4. Muscle memory doesn't build when you're context switching between OSes

## The Big Decision: Switching to Linux Full-Time
I've been dual-booting and using VMs, but it's slowing me down. Every time I switch back to Windows for "normal" work, I lose momentum. My hands forget the commands. The environment feels unfamiliar again.

So I'm making the switch. Linux as my daily driver. Not part-time, not in a VM. Full commitment.

If I'm serious about DevOps, I need to live in the environment I'm learning. No more excuses.

## Mistakes I Made
- Tried to SSH without specifying username, got confused why it wasn't working
- Forgot to use `sudo` for commands that needed elevated permissions
- Wrote a script with Windows line endings, spent 20 minutes debugging why bash couldn't read it

## Questions I Still Have
- What's the difference between SSH keys (RSA, ED25519, etc.)?
- How do you secure SSH properly in production?
- When do you use `curl` vs `wget`?
- What networking commands do actual sysadmins use daily?

## Tomorrow's Plan
- Set up Linux as primary OS
- Learn more about SSH security best practices
- Write more complex shell scripts with error handling
- Explore systemd and service management

---

**Progress: 6/90 days complete**
