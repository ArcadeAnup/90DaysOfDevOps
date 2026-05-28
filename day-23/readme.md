# Day 23: Jenkins Setup, RBAC, Backup, and Monitoring

## Jenkins Installation
- Installed on VirtualBox Ubuntu VM
- Initial unlock via: `/var/lib/jenkins/secrets/initialAdminPassword`
- Installed recommended plugins

## Jenkins Filesystem
/var/lib/jenkins/
├── config.xml           # Main config
├── jobs/                # Job definitions
├── plugins/             # Installed plugins
├── secrets/             # Encryption keys
├── users/               # User configs
└── logs/

**Key:** Everything is files. Easy backups.

## Creating First Freestyle Job

1. New Item → Freestyle job
2. Build section → Execute shell
3. Enter commands:
```bash
echo "Hello from Jenkins"
date
whoami
```
4. Save → Build Now
5. View Console Output

## Role-Based Access Control (RBAC)

**Install Plugin:** Manage Jenkins → Manage Plugins → Role-based Authorization Strategy

**Create Roles:**
Admin Role: administer, create, delete, configure, read
Developer Role: build, read, extended_read (cannot modify configs)

**Create Users:**
1. Manage Jenkins → Manage Users → Create user
2. Assign Roles → Select user → Check role

## Backup and Restore

**Install ThinBackup Plugin**

**Create Backup:**
```bash
sudo mkdir -p /backups/jenkins
sudo chown jenkins:jenkins /backups/jenkins
```

In UI: Manage Jenkins → ThinBackup → Backup Now

**Restore:**
1. Manage Jenkins → ThinBackup → Restore
2. Select backup date
3. Click Restore (Jenkins restarts automatically)

## Basic Monitoring

**Check Jenkins Status:**
```bash
sudo systemctl status jenkins
sudo tail -f /var/log/jenkins/jenkins.log
df -h          # Disk space
free -h        # Memory
ps aux | grep jenkins
```

**Monitor:**
- Disk space (builds + artifacts)
- Memory usage (plugins)
- Build queue
- Executor availability

## What Jenkins Does
Code Change → Jenkins detects → Runs build/test →
Reports results → Deploys (if configured)

## Commands Summary

```bash
# Backup
docker exec jenkins java -jar cli.jar backup

# Restore
docker exec jenkins java -jar cli.jar restore

# Check logs
sudo tail -100 /var/log/jenkins/jenkins.log

# Manage Jenkins service
sudo systemctl start/stop/restart jenkins
```

## Key Takeaways
1. Freestyle jobs = shell commands on button click
2. RBAC = enterprise permission management
3. File-based storage = easy backups and recovery
4. ThinBackup = disaster recovery solution
5. Jenkins centralizes all automation



**Progress: 23/90 days complete**