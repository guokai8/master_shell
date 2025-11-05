# Chapter 3: Command Not Found Errors
## Solving PATH and Command Execution Issues

---

## Overview

"Command not found" is one of the most frustrating Bash errors for beginners, yet it has straightforward causes and solutions. This chapter teaches you to diagnose and fix command execution issues on both MacOS and Linux.

By the end of this chapter, you'll master:
- Understanding the PATH variable
- Installing missing commands
- Fixing command availability issues
- Handling custom scripts and executables
- Platform-specific command differences

---

## 3.1 Understanding PATH

### What is PATH?

PATH is an environment variable that tells Bash where to look for executable commands.

```bash
# View your PATH
echo $PATH

# Sample output:
# /usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

**How it works:**
- Bash searches directories in order (left to right)
- First match is executed
- If not found in any directory, you get "command not found"

---

### Viewing PATH Clearly

```bash
#!/bin/bash
# show_path.sh

echo "Your PATH directories:"
echo "===================="
echo "$PATH" | tr ':' '\n' | nl
```

**Output:**
```
Your PATH directories:
====================
     1	/usr/local/bin
     2	/usr/bin
     3	/bin
     4	/usr/sbin
     5	/sbin
```

---

## 3.2 Common "Command Not Found" Scenarios

### Scenario 1: Command Not Installed

**Error:**
```bash
$ python3
bash: python3: command not found
```

**Diagnosis:**
```bash
# Check if installed
which python3
type python3
command -v python3
```

**Solution:**

**MacOS:**
```bash
# Install via Homebrew
brew install python3

# Or use system Python 3
python3 --version  # Usually pre-installed
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install python3
```

**Linux (CentOS/RHEL):**
```bash
sudo yum install python3
```

---

### Scenario 2: Command in Wrong Location

**Error:**
```bash
$ myscript.sh
bash: myscript.sh: command not found
```

**Diagnosis:**
```bash
# Check if file exists
ls -l myscript.sh

# Check current directory
pwd

# Try with explicit path
./myscript.sh
```

**Solution:**
```bash
# Use ./ for current directory
./myscript.sh

# Or add to PATH
export PATH="$PATH:$(pwd)"

# Or use absolute path
/full/path/to/myscript.sh
```

---

### Scenario 3: Permission Issues

**Error:**
```bash
$ ./script.sh
bash: ./script.sh: Permission denied
```

**Diagnosis:**
```bash
# Check permissions
ls -l script.sh

# Output shows: -rw-r--r-- (not executable)
```

**Solution:**
```bash
# Make executable
chmod +x script.sh

# Now run it
./script.sh
```

---

### Scenario 4: Wrong PATH

**Error:**
```bash
$ mycommand
bash: mycommand: command not found
```

**Diagnosis:**
```bash
# Find where command is
find /usr -name mycommand 2>/dev/null

# Found at: /usr/local/sbin/mycommand
# But /usr/local/sbin is not in PATH
```

**Solution:**
```bash
# Temporary fix
export PATH="$PATH:/usr/local/sbin"

# Permanent fix (add to ~/.bashrc)
echo 'export PATH="$PATH:/usr/local/sbin"' >> ~/.bashrc
source ~/.bashrc
```

---

## 3.3 Platform-Specific Commands

### MacOS vs Linux Differences

#### Package Managers

**MacOS:**
```bash
# Homebrew
brew install package

# MacPorts (alternative)
sudo port install package
```

**Linux:**
```bash
# Debian/Ubuntu
sudo apt install package

# RHEL/CentOS
sudo yum install package

# Fedora
sudo dnf install package

# Arch
sudo pacman -S package
```

---

#### Command Locations

**MacOS Homebrew paths:**
```bash
# Intel Macs
/usr/local/bin
/usr/local/sbin

# Apple Silicon (M1/M2)
/opt/homebrew/bin
/opt/homebrew/sbin
```

**Linux standard paths:**
```bash
/bin          # Essential commands
/usr/bin      # User commands
/usr/local/bin # Locally installed
/sbin         # System commands
/usr/sbin     # System commands
```

---

### Finding Missing Commands

#### Search Script

```bash
#!/bin/bash
# find_command.sh

command_name="$1"

if [ -z "$command_name" ]; then
    echo "Usage: $0 <command_name>"
    exit 1
fi

echo "Searching for: $command_name"
echo "======================"
echo

# Check if in PATH
if command -v "$command_name" &>/dev/null; then
    echo "✓ Found in PATH:"
    which "$command_name"
    echo
fi

# Search in common locations
echo "Searching system..."
locations=(
    "/bin"
    "/usr/bin"
    "/usr/local/bin"
    "/opt/homebrew/bin"
    "/sbin"
    "/usr/sbin"
)

for loc in "${locations[@]}"; do
    if [ -f "$loc/$command_name" ]; then
        echo "✓ Found: $loc/$command_name"
    fi
done

# Search everywhere (may be slow)
echo
echo "Full system search..."
find /usr /opt -name "$command_name" -type f 2>/dev/null | head -5
```

---

## 3.4 Managing Custom Scripts

### Installing Custom Scripts

**Step 1: Create a bin directory**
```bash
mkdir -p ~/bin
```

**Step 2: Add to PATH**
```bash
# Add to ~/.bashrc
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

**Step 3: Move or link your scripts**
```bash
# Move script
mv myscript.sh ~/bin/

# Or create symlink
ln -s /full/path/to/myscript.sh ~/bin/myscript

# Make executable
chmod +x ~/bin/myscript.sh
```

**Step 4: Use from anywhere**
```bash
# Now works from any directory
myscript.sh
```

---

### Script Template

```bash
#!/bin/bash
# template.sh

set -euo pipefail

# Script metadata
SCRIPT_NAME=$(basename "$0")
SCRIPT_DIR=$(cd "$(dirname "$0")" && pwd)

# Help function
show_help() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS] <argument>

Description of what this script does.

OPTIONS:
    -h, --help      Show this help message
    -v, --verbose   Enable verbose output

EXAMPLES:
    $SCRIPT_NAME file.txt
    $SCRIPT_NAME -v file.txt
EOF
}

# Main function
main() {
    # Your code here
    echo "Script running..."
}

# Parse arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            show_help
            exit 0
            ;;
        -v|--verbose)
            VERBOSE=true
            shift
            ;;
        *)
            ARGUMENT="$1"
            shift
            ;;
    esac
done

# Run main function
main
```

---

## 3.5 Troubleshooting PATH Issues

### PATH Diagnostic Script

```bash
#!/bin/bash
# diagnose_path.sh

echo "PATH Diagnostics"
echo "================"
echo

# Show current PATH
echo "Current PATH:"
echo "$PATH" | tr ':' '\n' | nl
echo

# Check each directory
echo "Checking PATH directories:"
echo "--------------------------"

IFS=':' read -ra DIRS <<< "$PATH"
for dir in "${DIRS[@]}"; do
    if [ -d "$dir" ]; then
        count=$(ls -1 "$dir" 2>/dev/null | wc -l)
        echo "✓ $dir ($count commands)"
    else
        echo "✗ $dir (doesn't exist)"
    fi
done
echo

# Find duplicates
echo "Checking for duplicates:"
echo "------------------------"
echo "$PATH" | tr ':' '\n' | sort | uniq -d | while read dup; do
    [ -n "$dup" ] && echo "⚠ Duplicate: $dup"
done

# Check common missing directories
echo
echo "Common directories not in PATH:"
echo "--------------------------------"
common_dirs=(
    "$HOME/bin"
    "$HOME/.local/bin"
    "/usr/local/bin"
    "/opt/homebrew/bin"
)

for dir in "${common_dirs[@]}"; do
    if [ -d "$dir" ] && [[ ":$PATH:" != *":$dir:"* ]]; then
        echo "⚠ $dir exists but not in PATH"
    fi
done
```

---

### Fixing PATH Issues

```bash
#!/bin/bash
# fix_path.sh

# Backup current PATH
echo "Current PATH:"
echo "$PATH"
echo

# Clean and rebuild PATH
clean_path() {
    # Remove duplicates while preserving order
    NEW_PATH=$(echo "$PATH" | tr ':' '\n' | awk '!seen[$0]++' | tr '\n' ':' | sed 's/:$//')
    echo "Cleaned PATH (duplicates removed):"
    echo "$NEW_PATH"
}

# Add directory to PATH if not present
add_to_path() {
    local dir="$1"

    if [ -d "$dir" ] && [[ ":$PATH:" != *":$dir:"* ]]; then
        export PATH="$dir:$PATH"
        echo "✓ Added $dir to PATH"
    else
        echo "⊘ $dir already in PATH or doesn't exist"
    fi
}

# Recommend PATH additions
echo "Recommended additions:"
echo "----------------------"
add_to_path "$HOME/bin"
add_to_path "$HOME/.local/bin"

if [[ "$OSTYPE" == "darwin"* ]]; then
    # MacOS
    add_to_path "/opt/homebrew/bin"
    add_to_path "/usr/local/bin"
fi

echo
echo "To make permanent, add to ~/.bashrc:"
echo "export PATH=\"$PATH\""
```

---

## 3.6 Command Alternatives and Aliases

### Finding Alternative Commands

```bash
#!/bin/bash
# find_alternatives.sh

command="$1"

echo "Finding alternatives for: $command"
echo "=================================="
echo

# Platform detection
if [[ "$OSTYPE" == "darwin"* ]]; then
    echo "Platform: MacOS"
    case "$command" in
        ll)
            echo "Alias suggestion:"
            echo "  alias ll='ls -lah'"
            ;;
        python)
            echo "Use: python3"
            which python3
            ;;
        sed)
            echo "MacOS uses BSD sed"
            echo "For GNU sed: brew install gnu-sed"
            ;;
    esac
else
    echo "Platform: Linux"
    case "$command" in
        ll)
            echo "May need alias:"
            echo "  alias ll='ls -lah'"
            ;;
        open)
            echo "Use: xdg-open"
            ;;
    esac
fi
```

---

### Creating Useful Aliases

Add these to `~/.bashrc`:

```bash
# Navigation aliases
alias ..='cd ..'
alias ...='cd ../..'
alias ll='ls -lah'
alias la='ls -A'

# Safety aliases
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Shortcuts
alias h='history'
alias j='jobs'
alias c='clear'

# Git aliases
alias gs='git status'
alias ga='git add'
alias gc='git commit'

# Platform-specific
if [[ "$OSTYPE" == "darwin"* ]]; then
    alias updatebrew='brew update && brew upgrade'
else
    alias update='sudo apt update && sudo apt upgrade'
fi

# Functions as aliases
# Extract any archive
extract() {
    if [ -f "$1" ]; then
        case "$1" in
            *.tar.bz2)   tar xjf "$1"    ;;
            *.tar.gz)    tar xzf "$1"    ;;
            *.bz2)       bunzip2 "$1"    ;;
            *.rar)       unrar x "$1"    ;;
            *.gz)        gunzip "$1"     ;;
            *.tar)       tar xf "$1"     ;;
            *.tbz2)      tar xjf "$1"    ;;
            *.tgz)       tar xzf "$1"    ;;
            *.zip)       unzip "$1"      ;;
            *.Z)         uncompress "$1" ;;
            *.7z)        7z x "$1"       ;;
            *)           echo "'$1' cannot be extracted" ;;
        esac
    else
        echo "'$1' is not a valid file"
    fi
}
```

---

## 3.7 Installing Common Commands

### Developer Tools

**MacOS:**
```bash
# Install Xcode Command Line Tools
xcode-select --install

# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Essential commands
brew install git wget curl tree jq
```

**Linux:**
```bash
# Ubuntu/Debian
sudo apt install build-essential git wget curl tree jq

# CentOS/RHEL
sudo yum groupinstall "Development Tools"
sudo yum install git wget curl tree jq
```

---

### Quick Install Reference

| Command | MacOS | Ubuntu/Debian | CentOS/RHEL |
|---------|-------|---------------|-------------|
| git | `brew install git` | `apt install git` | `yum install git` |
| python3 | `brew install python3` | `apt install python3` | `yum install python3` |
| node | `brew install node` | `apt install nodejs` | `yum install nodejs` |
| docker | `brew install docker` | `apt install docker.io` | `yum install docker` |
| vim | `brew install vim` | `apt install vim` | `yum install vim` |

---

## 3.8 Key Takeaways

✓ **Understand PATH** - Know where Bash looks for commands

✓ **Use ./ for local scripts** - Current directory not in PATH by default

✓ **Make scripts executable** - chmod +x before running

✓ **Add ~/bin to PATH** - For custom scripts

✓ **Check with which** - Find command locations

✓ **Platform differences** - MacOS and Linux have different commands

✓ **Install via package managers** - brew (MacOS), apt/yum (Linux)

---

## 3.9 Quick Reference

```bash
# Check if command exists
which command
command -v command
type command

# Check PATH
echo $PATH
echo $PATH | tr ':' '\n'

# Add to PATH (temporary)
export PATH="$PATH:/new/dir"

# Add to PATH (permanent)
echo 'export PATH="$PATH:/new/dir"' >> ~/.bashrc

# Run local script
./script.sh

# Make executable
chmod +x script.sh

# Find command location
find /usr -name command 2>/dev/null
```

---

*End of Chapter 3*

**Next:** [Chapter 4: Permission and Access Errors](chapter-04-permission-errors.md)
