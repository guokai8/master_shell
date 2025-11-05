# Chapter 1: Installation and Environment Setup
## Getting Started with Bash on MacOS and Linux

---

## Overview

Before diving into Bash error troubleshooting, you need a properly configured environment. This chapter guides you through installing and setting up Bash on both MacOS and Linux systems, ensuring you have the right tools and configuration for effective debugging.

By the end of this chapter, you'll have:
- Bash installed and updated on your system
- A properly configured shell environment
- Essential debugging tools installed
- Knowledge of your system's Bash version and capabilities

---

## 1.1 Understanding Bash Versions

### What is Bash?

Bash (Bourne Again Shell) is the default shell on most Linux distributions and was the default on MacOS until Catalina (10.15). Understanding your Bash version is crucial because:

- Different versions have different features
- Error messages may vary between versions
- Some syntax is version-specific

### Checking Your Bash Version

```bash
# Check Bash version
bash --version

# Alternative: Check current shell
echo $BASH_VERSION

# Check default shell
echo $SHELL

# Check available shells
cat /etc/shells
```

**Sample Output:**
```
GNU bash, version 5.2.15(1)-release (x86_64-apple-darwin23.0.0)
```

---

## 1.2 Installing Bash on MacOS

### Default Bash on MacOS

MacOS comes with Bash pre-installed, but it's often an older version due to licensing restrictions.

**Check your current version:**
```bash
bash --version
```

MacOS Monterey and later typically ship with Bash 3.2 (from 2007) due to GPLv2 vs GPLv3 licensing.

### Installing Updated Bash via Homebrew

**Step 1: Install Homebrew (if not already installed)**
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Step 2: Install latest Bash**
```bash
brew install bash
```

**Step 3: Verify installation**
```bash
/usr/local/bin/bash --version
# or on Apple Silicon:
/opt/homebrew/bin/bash --version
```

**Step 4: Add new Bash to allowed shells**
```bash
# Add to /etc/shells
echo "/usr/local/bin/bash" | sudo tee -a /etc/shells
# or on Apple Silicon:
echo "/opt/homebrew/bin/bash" | sudo tee -a /etc/shells
```

**Step 5: Change default shell (optional)**
```bash
chsh -s /usr/local/bin/bash
# or on Apple Silicon:
chsh -s /opt/homebrew/bin/bash
```

### MacOS-Specific Considerations

**Zsh is now default:**
Since MacOS Catalina (10.15), the default shell is Zsh. To switch to Bash:

```bash
# Temporary (current session only)
bash

# Permanent
chsh -s /bin/bash
```

---

## 1.3 Installing Bash on Linux

### Checking Current Installation

Most Linux distributions come with Bash pre-installed:

```bash
# Check version
bash --version

# Check if it's your default shell
echo $SHELL

# Verify it's in your PATH
which bash
```

### Installing/Updating Bash

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install bash
```

**CentOS/RHEL/Fedora:**
```bash
sudo yum install bash
# or
sudo dnf install bash
```

**Arch Linux:**
```bash
sudo pacman -S bash
```

### Building from Source (Advanced)

If you need the absolute latest version:

```bash
# Download latest Bash
wget https://ftp.gnu.org/gnu/bash/bash-5.2.tar.gz

# Extract
tar -xzvf bash-5.2.tar.gz
cd bash-5.2

# Configure and build
./configure
make
sudo make install
```

---

## 1.4 Essential Configuration Files

### Understanding Bash Configuration

Bash reads different configuration files depending on how it's invoked:

**Login Shell:**
- `/etc/profile` (system-wide)
- `~/.bash_profile`
- `~/.bash_login`
- `~/.profile`

**Non-Login Interactive Shell:**
- `/etc/bash.bashrc` (system-wide)
- `~/.bashrc`

**Non-Interactive Shell:**
- Uses `$BASH_ENV` variable

### Setting Up .bashrc

Create or edit `~/.bashrc`:

```bash
# ~/.bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
    . /etc/bashrc
fi

# User specific aliases and functions
alias ll='ls -lah'
alias grep='grep --color=auto'

# Set default editor
export EDITOR=nano

# Add custom directories to PATH
export PATH="$HOME/bin:$PATH"

# Enable bash completion
if [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
fi

# History settings
export HISTSIZE=10000
export HISTFILESIZE=20000
export HISTCONTROL=ignoredups:erasedups

# Prompt customization
PS1='\[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\]\$ '
```

### Setting Up .bash_profile

Create or edit `~/.bash_profile`:

```bash
# ~/.bash_profile

# Source .bashrc for login shells
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi

# MacOS specific: Add Homebrew to PATH
if [ -f /opt/homebrew/bin/brew ]; then
    eval "$(/opt/homebrew/bin/brew shellenv)"
fi
```

---

## 1.5 Essential Tools for Debugging

### Installing ShellCheck

ShellCheck is a static analysis tool for shell scripts:

**MacOS:**
```bash
brew install shellcheck
```

**Ubuntu/Debian:**
```bash
sudo apt install shellcheck
```

**Usage:**
```bash
shellcheck script.sh
```

### Installing bash-completion

Improves command and filename completion:

**MacOS:**
```bash
brew install bash-completion@2
```

**Ubuntu/Debian:**
```bash
sudo apt install bash-completion
```

### Other Useful Tools

```bash
# MacOS
brew install tree      # Directory tree viewer
brew install tldr      # Simplified man pages
brew install jq        # JSON processor

# Ubuntu/Debian
sudo apt install tree tldr jq
```

---

## 1.6 Environment Variables Setup

### Critical Environment Variables

```bash
# View current environment
printenv

# Essential variables to set
export PATH="$HOME/bin:/usr/local/bin:$PATH"
export EDITOR=nano
export VISUAL=nano
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

### PATH Management

```bash
# View current PATH
echo $PATH

# Add directory to PATH (temporary)
export PATH="/new/directory:$PATH"

# Add to PATH permanently (add to ~/.bashrc)
echo 'export PATH="/new/directory:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## 1.7 Verifying Your Setup

### Quick Verification Script

Save this as `verify_bash_setup.sh`:

```bash
#!/bin/bash

echo "Bash Environment Verification"
echo "=============================="
echo

echo "1. Bash Version:"
bash --version | head -1
echo

echo "2. Current Shell:"
echo $SHELL
echo

echo "3. Bash Location:"
which bash
echo

echo "4. PATH:"
echo $PATH | tr ':' '\n' | head -5
echo "   ... (showing first 5 entries)"
echo

echo "5. Essential Tools:"
for tool in shellcheck git nano; do
    if command -v $tool &> /dev/null; then
        echo "   ✓ $tool installed"
    else
        echo "   ✗ $tool not found"
    fi
done
echo

echo "6. Configuration Files:"
for file in ~/.bashrc ~/.bash_profile ~/.profile; do
    if [ -f $file ]; then
        echo "   ✓ $file exists"
    else
        echo "   ○ $file not found"
    fi
done
echo

echo "Setup verification complete!"
```

Run it:
```bash
chmod +x verify_bash_setup.sh
./verify_bash_setup.sh
```

---

## 1.8 Common Setup Issues

### Issue 1: Command Not Found

**Symptom:**
```
bash: command: command not found
```

**Solutions:**
```bash
# Check if command exists
which command

# Check PATH
echo $PATH

# Add to PATH
export PATH="/path/to/directory:$PATH"
```

### Issue 2: Configuration Not Loading

**Symptom:**
Changes to `.bashrc` don't take effect

**Solutions:**
```bash
# Reload configuration
source ~/.bashrc

# Check which file is being loaded
echo $BASH_SOURCE

# For login shells, edit .bash_profile instead
```

### Issue 3: Permission Denied

**Symptom:**
```
bash: ./script.sh: Permission denied
```

**Solution:**
```bash
chmod +x script.sh
```

---

## 1.9 Platform-Specific Notes

### MacOS Specifics

```bash
# Apple Silicon (M1/M2) Homebrew location
/opt/homebrew/bin

# Intel Mac Homebrew location
/usr/local/bin

# System Bash location
/bin/bash

# Check architecture
uname -m  # arm64 (Apple Silicon) or x86_64 (Intel)
```

### Linux Specifics

```bash
# Find bash location
which bash
type bash

# Check if bash is login shell
echo $0  # Starts with - if login shell

# Debian/Ubuntu specific
dpkg -l | grep bash

# RedHat/CentOS specific
rpm -qa | grep bash
```

---

## 1.10 Key Takeaways

✓ **Know your version** - Different Bash versions behave differently

✓ **Update when possible** - Newer versions have better error messages

✓ **Configure properly** - Set up .bashrc and .bash_profile correctly

✓ **Install tools** - ShellCheck and completion improve productivity

✓ **Verify setup** - Test your environment before proceeding

✓ **Document differences** - Note platform-specific behaviors

---

## 1.11 Quick Reference

```bash
# Version info
bash --version          # Full version
echo $BASH_VERSION      # Version variable
echo $SHELL            # Default shell

# Configuration
~/.bashrc              # Interactive non-login
~/.bash_profile        # Login shell
source ~/.bashrc       # Reload config

# Tools
shellcheck script.sh   # Check syntax
which command          # Find command location
type command           # Command info

# PATH
echo $PATH             # View PATH
export PATH="dir:$PATH" # Add to PATH
```

---

*End of Chapter 1*

**Next:** [Chapter 2: Understanding Error Messages](chapter-02-understanding-errors.md)
