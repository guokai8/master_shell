# Chapter 4: Permission and Access Errors
## Mastering File Permissions, Ownership, and Security

---

## Overview

Permission errors are among the most common obstacles in Bash scripting. Understanding Unix/Linux permissions is crucial for both MacOS and Linux users. This chapter demystifies file permissions, ownership, and security contexts.

By the end of this chapter, you'll master:
- Reading and setting file permissions
- Understanding ownership (user, group)
- Using chmod, chown, and chgrp effectively
- Handling sudo and privilege escalation
- Dealing with special permissions (setuid, setgid, sticky bit)
- Platform-specific permission systems (SELinux, AppArmor)

---

## 4.1 Understanding Unix Permissions

### The Permission Model

Every file and directory has three permission sets:

```
-rwxr-xr-x
│││││││││
│││││││└└─ Other (everyone else)
││││││└───┘
││││└└───── Group
│││└───────┘
││└──────── Owner
│└───────── Type (- file, d directory, l link)
└────────── Special permissions
```

**Permission Types:**
- `r` (read) = 4
- `w` (write) = 2
- `x` (execute) = 1
- `-` (no permission) = 0

---

### Reading Permissions

```bash
#!/bin/bash
# read_permissions.sh

explain_permissions() {
    local file="$1"

    if [[ ! -e "$file" ]]; then
        echo "File not found: $file"
        return 1
    fi

    # Get permissions in long format
    local perms=$(ls -ld "$file")

    echo "File: $file"
    echo "Permissions: $perms"
    echo

    # Extract permission string
    local perm_str=$(echo "$perms" | awk '{print $1}')

    echo "Breakdown:"
    echo "  Type: ${perm_str:0:1}"
    echo "  Owner: ${perm_str:1:3}"
    echo "  Group: ${perm_str:4:3}"
    echo "  Other: ${perm_str:7:3}"
    echo

    # Numeric representation
    local owner_num=$(( (${perm_str:1:1} == 'r' ? 4 : 0) + (${perm_str:2:1} == 'w' ? 2 : 0) + (${perm_str:3:1} == 'x' ? 1 : 0) ))
    local group_num=$(( (${perm_str:4:1} == 'r' ? 4 : 0) + (${perm_str:5:1} == 'w' ? 2 : 0) + (${perm_str:6:1} == 'x' ? 1 : 0) ))
    local other_num=$(( (${perm_str:7:1} == 'r' ? 4 : 0) + (${perm_str:8:1} == 'w' ? 2 : 0) + (${perm_str:9:1} == 'x' ? 1 : 0) ))

    echo "Numeric: $owner_num$group_num$other_num"
}

# Test
explain_permissions "$1"
```

---

## 4.2 Common Permission Errors

### Error 1: Permission Denied (Execution)

**Error:**
```bash
$ ./script.sh
bash: ./script.sh: Permission denied
```

**Diagnosis:**
```bash
ls -l script.sh
# Output: -rw-r--r--  (not executable)
```

**Solution:**
```bash
chmod +x script.sh
# or
chmod 755 script.sh
```

---

### Error 2: Permission Denied (Reading)

**Error:**
```bash
$ cat file.txt
cat: file.txt: Permission denied
```

**Diagnosis:**
```bash
ls -l file.txt
# Output: --w-------  (write only, no read)
```

**Solution:**
```bash
chmod +r file.txt
# or
chmod 644 file.txt
```

---

### Error 3: Permission Denied (Directory)

**Error:**
```bash
$ cd /restricted
bash: cd: /restricted: Permission denied
```

**Diagnosis:**
```bash
ls -ld /restricted
# Output: drw-------  (no execute on directory)
```

**Solution:**
```bash
# Directory execute permission needed to cd into it
chmod +x /restricted
# or
chmod 755 /restricted
```

---

## 4.3 chmod Command Deep Dive

### Symbolic Mode

```bash
# Format: chmod [who][op][permission] file

# Who:
u = user/owner
g = group
o = others
a = all (default)

# Operations:
+ = add permission
- = remove permission
= = set exact permission

# Permissions:
r = read
w = write
x = execute

# Examples:
chmod u+x script.sh        # Add execute for owner
chmod g-w file.txt         # Remove write for group
chmod o=r file.txt         # Set other to read only
chmod a+x script.sh        # Add execute for all
chmod u+rw,g+r file.txt    # Multiple operations
```

---

### Numeric Mode

```bash
# Format: chmod ### file
# Each digit is sum of: r(4) + w(2) + x(1)

chmod 777 file  # rwxrwxrwx (all permissions)
chmod 755 file  # rwxr-xr-x (owner full, others read+execute)
chmod 644 file  # rw-r--r-- (owner read+write, others read)
chmod 600 file  # rw------- (owner only)
chmod 700 dir   # rwx------ (private directory)
chmod 666 file  # rw-rw-rw- (all can read/write)
chmod 444 file  # r--r--r-- (read-only for all)
```

**Common Patterns:**
```bash
# Scripts
chmod 755 script.sh  # Executable by all, writable by owner

# Configuration files
chmod 644 config.ini # Readable by all, writable by owner

# Private files
chmod 600 secrets.txt # Owner only

# Directories
chmod 755 public_dir/    # Accessible by all
chmod 700 private_dir/   # Owner only
```

---

### Recursive chmod

```bash
# Apply to directory and all contents
chmod -R 755 directory/

# Apply different permissions to files and directories
find directory/ -type f -exec chmod 644 {} \;  # Files
find directory/ -type d -exec chmod 755 {} \;  # Directories
```

---

## 4.4 File Ownership

### Understanding chown

```bash
# View ownership
ls -l file.txt
# Output: -rw-r--r-- 1 alice developers 1234 Nov 1 10:00 file.txt
#                      │     │
#                      │     └─ Group
#                      └─ Owner

# Change owner
sudo chown bob file.txt

# Change owner and group
sudo chown bob:users file.txt

# Change group only
sudo chgrp users file.txt
# or
sudo chown :users file.txt

# Recursive
sudo chown -R bob:users directory/
```

---

### Ownership Troubleshooting Script

```bash
#!/bin/bash
# check_ownership.sh

file="$1"

if [[ ! -e "$file" ]]; then
    echo "File not found: $file"
    exit 1
fi

echo "Ownership Information"
echo "===================="
echo

# Get file info
owner=$(stat -c '%U' "$file" 2>/dev/null || stat -f '%Su' "$file")
group=$(stat -c '%G' "$file" 2>/dev/null || stat -f '%Sg' "$file")
perms=$(stat -c '%a' "$file" 2>/dev/null || stat -f '%Lp' "$file")

echo "File: $file"
echo "Owner: $owner"
echo "Group: $group"
echo "Permissions: $perms"
echo

# Check if current user can access
current_user=$(whoami)
user_groups=$(groups)

echo "Current user: $current_user"
echo "User's groups: $user_groups"
echo

# Determine access
if [[ "$owner" == "$current_user" ]]; then
    echo "✓ You own this file"
elif echo "$user_groups" | grep -q "$group"; then
    echo "✓ You are in the group ($group)"
else
    echo "✗ You don't own this file or belong to its group"
    echo "  You rely on 'other' permissions"
fi
```

---

## 4.5 Using sudo Properly

### When to Use sudo

```bash
# System files (require root)
sudo vim /etc/hosts
sudo systemctl restart service

# Installing software
sudo apt install package

# Changing ownership
sudo chown user:group file

# System directories
sudo mkdir /opt/myapp
sudo cp file /usr/local/bin/
```

---

### sudo Best Practices

```bash
#!/bin/bash
# sudo_best_practices.sh

# ✗ BAD: Running entire script as root
# sudo ./script.sh

# ✓ GOOD: Only elevate when needed
check_root_needed() {
    if [[ $EUID -eq 0 ]]; then
        echo "✗ Don't run entire script as root"
        exit 1
    fi
}

install_file() {
    local file="$1"
    local dest="$2"

    # Regular operations
    echo "Preparing file..."
    cp "$file" "/tmp/temp_file"

    # Only use sudo when necessary
    echo "Installing (requires sudo)..."
    sudo cp "/tmp/temp_file" "$dest"
    sudo chmod 644 "$dest"

    # Cleanup without sudo
    rm "/tmp/temp_file"
}

check_root_needed
```

---

### Checking sudo Access

```bash
#!/bin/bash
# check_sudo.sh

echo "Checking sudo access..."

# Check if user has sudo
if sudo -n true 2>/dev/null; then
    echo "✓ You have sudo access (no password needed)"
elif sudo -v 2>/dev/null; then
    echo "✓ You have sudo access"
else
    echo "✗ You don't have sudo access"
    echo "  Contact your administrator"
    exit 1
fi

# Show sudo privileges
echo
echo "Your sudo privileges:"
sudo -l 2>/dev/null | grep -v "may run" | head -5
```

---

## 4.6 Special Permissions

### Setuid, Setgid, and Sticky Bit

```bash
# Setuid (4000): Execute as file owner
chmod u+s file
chmod 4755 file
# Example: -rwsr-xr-x (note 's' instead of 'x')

# Setgid (2000): Execute as file's group
chmod g+s file
chmod 2755 file
# Example: -rwxr-sr-x

# Sticky bit (1000): Only owner can delete
chmod +t directory
chmod 1777 directory
# Example: drwxrwxrwt (note 't' at end)
```

**Common Uses:**
```bash
# /tmp directory (sticky bit)
ls -ld /tmp
# drwxrwxrwt  # Anyone can write, only owner can delete their files

# Shared directory with setgid
chmod 2775 /shared/project
# New files inherit group ownership
```

---

### Demonstration Script

```bash
#!/bin/bash
# demonstrate_special_perms.sh

demo_dir="/tmp/perms_demo_$$"
mkdir -p "$demo_dir"
cd "$demo_dir" || exit

echo "Special Permissions Demo"
echo "======================="
echo

# Regular directory
mkdir regular_dir
chmod 777 regular_dir
echo "1. Regular directory (777):"
ls -ld regular_dir
echo

# Sticky bit directory
mkdir sticky_dir
chmod 1777 sticky_dir
echo "2. Sticky bit directory (1777):"
ls -ld sticky_dir
echo "   Notice the 't' at the end"
echo "   Only file owner can delete their files"
echo

# Setgid directory
mkdir setgid_dir
chmod 2775 setgid_dir
echo "3. Setgid directory (2775):"
ls -ld setgid_dir
echo "   Notice the 's' in group permissions"
echo "   New files inherit group ownership"
echo

# Cleanup
cd /
rm -rf "$demo_dir"
```

---

## 4.7 MacOS-Specific Permissions

### System Integrity Protection (SIP)

MacOS has additional security:

```bash
# Check SIP status
csrutil status

# Protected directories:
/System
/usr (except /usr/local)
/bin
/sbin

# Even sudo can't modify these when SIP is enabled
```

---

### Extended Attributes (xattr)

MacOS uses extended attributes:

```bash
# List extended attributes
xattr file.txt
ls -l@ file.txt

# Common attribute: quarantine (downloaded files)
# Remove quarantine
xattr -d com.apple.quarantine file.txt

# Remove all extended attributes
xattr -c file.txt

# Recursively remove
xattr -cr directory/
```

---

### ACL (Access Control Lists)

```bash
# View ACLs
ls -le file.txt

# Add ACL
chmod +a "user:username allow read,write" file.txt

# Remove ACL
chmod -a "user:username allow read,write" file.txt

# Remove all ACLs
chmod -N file.txt
```

---

## 4.8 Linux Security Contexts: SELinux and AppArmor

### SELinux (CentOS/RHEL/Fedora)

```bash
# Check SELinux status
getenforce
# Output: Enforcing, Permissive, or Disabled

# View file context
ls -Z file.txt

# Common error with SELinux:
# Permission denied despite correct Unix permissions

# Fix context
restorecon -v file.txt

# Change context temporarily
chcon -t httpd_sys_content_t /var/www/html/index.html

# Make persistent
semanage fcontext -a -t httpd_sys_content_t "/var/www/html(/.*)?"
restorecon -Rv /var/www/html

# Temporarily disable (testing only!)
sudo setenforce 0  # Permissive mode
sudo setenforce 1  # Enforcing mode
```

---

### AppArmor (Ubuntu/Debian)

```bash
# Check AppArmor status
sudo aa-status

# View profile status
sudo aa-status | grep /usr/bin/program

# Set to complain mode (log but don't block)
sudo aa-complain /usr/bin/program

# Set to enforce mode
sudo aa-enforce /usr/bin/program

# Disable profile
sudo ln -s /etc/apparmor.d/usr.bin.program /etc/apparmor.d/disable/
sudo apparmor_parser -R /etc/apparmor.d/usr.bin.program

# Check denials
sudo grep DENIED /var/log/syslog
```

---

## 4.9 Comprehensive Permission Troubleshooter

```bash
#!/bin/bash
# permission_doctor.sh

target="$1"

if [[ -z "$target" ]]; then
    echo "Usage: $0 <file_or_directory>"
    exit 1
fi

echo "╔══════════════════════════════════════════════════════════╗"
echo "║         PERMISSION DIAGNOSTIC REPORT                     ║"
echo "╚══════════════════════════════════════════════════════════╝"
echo
echo "Target: $target"
echo "═══════════════════════════════════════════════════════════"
echo

# Check existence
if [[ ! -e "$target" ]]; then
    echo "✗ Target does not exist!"
    exit 1
fi

echo "✓ Target exists"
echo

# 1. Basic permissions
echo "1. PERMISSIONS"
echo "───────────────────────────────────────────────────────────"
ls -ld "$target"
echo

# 2. Ownership
owner=$(stat -c '%U' "$target" 2>/dev/null || stat -f '%Su' "$target")
group=$(stat -c '%G' "$target" 2>/dev/null || stat -f '%Sg' "$target")

echo "2. OWNERSHIP"
echo "───────────────────────────────────────────────────────────"
echo "Owner: $owner"
echo "Group: $group"
echo

if [[ "$(whoami)" == "$owner" ]]; then
    echo "✓ You own this file"
else
    echo "✗ You don't own this file (owner: $owner)"
fi

if groups | grep -q "\b$group\b"; then
    echo "✓ You are in the group ($group)"
else
    echo "✗ You are not in the group ($group)"
fi
echo

# 3. Your access
echo "3. YOUR ACCESS"
echo "───────────────────────────────────────────────────────────"
[[ -r "$target" ]] && echo "✓ Read" || echo "✗ Read"
[[ -w "$target" ]] && echo "✓ Write" || echo "✗ Write"
[[ -x "$target" ]] && echo "✓ Execute" || echo "✗ Execute"
echo

# 4. Parent directory
parent=$(dirname "$target")
echo "4. PARENT DIRECTORY ACCESS"
echo "───────────────────────────────────────────────────────────"
echo "Parent: $parent"
ls -ld "$parent"
echo

if [[ ! -x "$parent" ]]; then
    echo "⚠ WARNING: No execute permission on parent directory!"
    echo "   You need execute permission to access files inside"
fi
echo

# 5. Recommendations
echo "5. RECOMMENDATIONS"
echo "───────────────────────────────────────────────────────────"

if [[ ! -r "$target" ]]; then
    echo "To add read permission:"
    if [[ "$(whoami)" == "$owner" ]]; then
        echo "  chmod u+r $target"
    else
        echo "  sudo chmod o+r $target  # For everyone"
    fi
fi

if [[ ! -w "$target" ]]; then
    echo "To add write permission:"
    if [[ "$(whoami)" == "$owner" ]]; then
        echo "  chmod u+w $target"
    else
        echo "  sudo chmod o+w $target  # For everyone (risky)"
    fi
fi

if [[ -f "$target" ]] && [[ ! -x "$target" ]]; then
    echo "To make executable:"
    echo "  chmod +x $target"
fi

echo
echo "═══════════════════════════════════════════════════════════"
```

---

## 4.10 Key Takeaways

✓ **Understand the permission model** - Owner, Group, Other with Read, Write, Execute

✓ **Use chmod correctly** - Both numeric (755) and symbolic (u+x) modes

✓ **Know when to use sudo** - Only for system files and administrative tasks

✓ **Check ownership** - Use ls -l to see owner and group

✓ **Be careful with special permissions** - Setuid, setgid, sticky bit

✓ **MacOS has extra security** - SIP, extended attributes, ACLs

✓ **Linux has security contexts** - SELinux, AppArmor

✓ **Never use chmod 777** - Almost always a security risk

---

## 4.11 Quick Reference

```bash
# View permissions
ls -l file
ls -ld directory
stat file

# chmod (symbolic)
chmod u+x file      # Add execute for owner
chmod g-w file      # Remove write for group
chmod o=r file      # Set other to read-only
chmod a+r file      # Add read for all

# chmod (numeric)
chmod 755 file      # rwxr-xr-x
chmod 644 file      # rw-r--r--
chmod 600 file      # rw-------

# chown
sudo chown user file
sudo chown user:group file
sudo chgrp group file

# Special permissions
chmod u+s file      # Setuid
chmod g+s directory # Setgid
chmod +t directory  # Sticky bit

# Check access
[ -r file ] && echo "Readable"
[ -w file ] && echo "Writable"
[ -x file ] && echo "Executable"
```

---

*End of Chapter 4*

**Next:** [Chapter 5: File and Directory Errors](chapter-05-file-directory-errors.md)
