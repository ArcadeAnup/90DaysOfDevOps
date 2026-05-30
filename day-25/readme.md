
## Azure Student Account Setup
- Activated Azure Student Benefits: $100 free credits
- Set up Azure Portal account
- No payment required (for $100 credit period)

## Created First Cloud VM

**Steps:**
1. Azure Portal → Virtual Machines → Create
2. Selected: Ubuntu 22.04 LTS
3. Size: Standard B1s (cheap, good for learning)
4. Authentication: SSH key pair (generated in portal)
5. Created in ~2 minutes

**Result:** Real public IP address assigned automatically

## SSH'd Into Cloud VM

```bash
# Download private key from Azure Portal
# Set permissions
chmod 600 key.pem

# SSH into VM
ssh -i key.pem azureuser@
```

**Now working on actual Ubuntu in the cloud**, not VirtualBox.

## Opened Firewall Rules (NSG)

Azure NSG (Network Security Group) = firewall rules.

**Opened Ports:**
- Port 22: SSH access
- Port 80: HTTP (Nginx)
- Port 8080: Jenkins
Azure Portal → VM → Networking → Add Inbound Rule
Protocol: TCP, Port: 80, Source: Any, Action: Allow

## Ran Nginx (Web Server)

```bash
# Install
sudo apt-get update
sudo apt-get install nginx

# Start
sudo systemctl start nginx

# Verify
curl localhost

# Access from internet
# Go to: http://<public-ip>
# See Nginx welcome page ✅
```

**Real web server. Public internet. Working.**

## Installed Jenkins on Azure

```bash
# Install Java (Jenkins requirement)
sudo apt-get install default-jre

# Add Jenkins repository and install
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo apt-get install jenkins

# Start Jenkins
sudo systemctl start jenkins

# Access
# http://<public-ip>:8080
```

**Initial unlock password:**
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Set up admin user, installed plugins, ready to use.

## GitHub Actions Continued

**Parallel workflow:**
- Azure: Jenkins running publicly
- GitHub: CI pipeline running automatically

**Matrix Builds:**
```yaml
matrix:
  node-version: [18.x, 20.x, 22.x]
```

**Result:**
- Pushed code
- GitHub Actions triggered automatically
- Built and tested on Node 18, 20, 22
- All passed ✅
- Merged PR

## What This Means
Local Development:
Write code → Test locally → Push
With Cloud + CI/CD:
Write code → Push → GitHub Actions runs tests →
If pass: PR ready to merge →
Jenkins ready to deploy (from Azure VM)

Entire automated pipeline. No manual intervention.

## Why This Matters

**Before (Days 1-24):**
- Learning tools in isolation
- Building locally
- No real deployment

**After (Day 25):**
- Real cloud infrastructure
- Real CI/CD pipeline
- Code → Test → Deploy workflow
- Public deployment

This is actual DevOps workflow, not simulated.

## Quick Wins
1. Azure student credits = free cloud lab
2. SSH into cloud VM = real infrastructure
3. Open firewall ports = expose services publicly
4. Jenkins + GitHub Actions = full CI/CD
5. Real public IP = actually deployed to internet

## Key Learnings

**Cloud > Local Infrastructure:**
- No resource constraints (VirtualBox limited)
- Realistic environment (actual Ubuntu server)
- Scalable (add more VMs if needed)
- Cost-effective for learning (free credits)

**CI/CD in Practice:**
- GitHub Actions automatically tests on push
- Matrix builds = test multiple environments
- Jenkins ready for deployment from Azure
- Fully automated pipeline


## Commands Summary

```bash
# SSH into Azure VM
ssh -i key.pem azureuser@<public-ip>

# Start/stop services
sudo systemctl start/stop/restart nginx
sudo systemctl start/stop/restart jenkins

# Check service status
sudo systemctl status jenkins

# View Jenkins logs
sudo tail -f /var/log/jenkins/jenkins.log

# Get Jenkins initial password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```


---

**Progress: 25/90 days complete**
