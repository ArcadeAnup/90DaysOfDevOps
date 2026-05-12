# Day 5: Shell Scripting, Pipes, Redirects, and Vim

## What I Learned Today

### My First Bash Script
- Wrote my first working bash script to automate repetitive commands
- Basic structure:
```bash
#!/bin/bash
# This is a comment
echo "Starting script..."
# Your commands here
```
- `#!/bin/bash` is the shebang - tells the system to use bash to run this script
- Make scripts executable with `chmod +x script.sh`
- Run with `./script.sh`

### Pipes and Redirects

#### Pipes (`|`)
- Takes output from one command and feeds it as input to another
- Example: `ls -l | grep "txt"` - lists files, then filters for .txt files
- You can chain multiple pipes: `command1 | command2 | command3`
- Super useful for filtering and processing data on the fly

#### Redirects
- `>` - redirects output to a file (overwrites existing content)
  - Example: `echo "hello" > file.txt`
- `>>` - appends output to a file (doesn't overwrite)
  - Example: `echo "world" >> file.txt`
- `<` - takes input from a file
  - Example: `sort < unsorted.txt`
- `2>` - redirects error messages
  - Example: `command 2> errors.log`
- `&>` - redirects both output and errors
  - Example: `command &> all_output.log`

### Vim Editor
- Vim is a powerful text editor that lives in your terminal
- Has different modes:
  - **Normal mode**: Default mode, navigate and run commands
  - **Insert mode**: Actually type text (press `i` to enter)
  - **Command mode**: Save, quit, search (press `:` to enter)

#### Basic Vim Commands
- `i` - enter insert mode
- `Esc` - back to normal mode
- `:w` - save (write)
- `:q` - quit
- `:wq` - save and quit
- `:q!` - quit without saving
- `dd` - delete line
- `yy` - copy line
- `p` - paste
- `/searchterm` - search for text
- `u` - undo

Why vim? It's installed on basically every Linux system, works over SSH, and once you learn it, you're fast. The learning curve is brutal though.

### Revision: Day 3 and 4 Topics
Circled back to concepts from previous days:

#### File Permissions (Day 4)
- `rwxrwxrwx` = owner, group, others
- `chmod 755 file` = owner gets rwx (7), group and others get rx (5)
- Now makes way more sense after using it in practice

#### Linux File System (Day 3)
- `/bin` - essential binaries
- `/etc` - configuration files
- `/home` - user directories
- `/var` - variable data (logs, temp files)
- Feels less intimidating after actually navigating it

#### SDLC and VMs (Day 3)
- Virtual Machines let you run isolated environments
- SDLC phases: plan, design, develop, test, deploy, maintain
- Makes more sense now that I'm actually deploying things on EC2

## What I Built Today
A basic bash script that:
1. Creates a directory structure
2. Sets proper permissions
3. Logs the output to a file
4. Displays a success message

It's not fancy, but it works and saves me from typing the same 10 commands every time.

## Key Takeaways
1. Shell scripts are just commands you'd type manually, saved in a file
2. Pipes let you chain commands together like Lego blocks
3. Redirects control where your output goes
4. Vim has a steep learning curve but it's worth learning the basics
5. Revisiting old topics makes them stick better

## Mistakes I Made
- Forgot to make my script executable and wondered why it wouldn't run
- Spent way too long stuck in vim before remembering `:q!`
- Used `>` instead of `>>` and accidentally wiped a file (oops)


**Progress: 5/90 days complete**