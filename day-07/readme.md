# Day 7: AWS Infrastructure Audit Script and Linux Practice

## What I Learned Today

### AWS CLI (Command Line Interface)
- The AWS CLI lets you interact with AWS services directly from the terminal
- Install: `pip install awscli` or use package manager
- Configure: `aws configure` (need Access Key ID and Secret Access Key)
- Much faster than clicking through the AWS console for routine checks



### The AWS Infrastructure Audit Script

Built a shell script that audits the entire AWS setup. Here's what it does:

#### Features
1. Lists all S3 buckets in the account
2. Shows EC2 instances with their status and type
3. Displays IAM users (for security auditing)
4. Lists Lambda functions
5. Shows current AWS account details
6. Outputs everything in a readable format
7. Can redirect output to a log file for record-keeping


#### How to Use
```bash
# Make it executable
chmod +x aws_audit.sh

# Run it
./aws_audit.sh

```

### Linux Quizzes
Solved quizzes on:
- File permissions and chmod
- Pipes and redirects
- Basic shell scripting
- Networking commands
- SSH and remote connections

Tested what actually stuck vs what I just memorized temporarily. Retained way more than expected, which means the hands-on practice is working.

## What I Built Today
A functional AWS infrastructure audit script that:
- Runs without errors
- Outputs readable information
- Can be scheduled with cron for regular audits
- Helped me find resources I forgot I was running 



## Real-World Use Cases for This Script
- Daily/weekly infrastructure audits
- Cost monitoring (see what's running and where money's going)
- Security checks (who has IAM access?)
- Documenting infrastructure for compliance
- Quick overview before making changes

##