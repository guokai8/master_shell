# Chapter 2: Understanding Error Messages
## Decoding Bash Error Output

---

## Overview

Understanding error messages is the first step in effective troubleshooting. Bash error messages follow patterns, and learning to decode them quickly will save you hours of debugging. This chapter teaches you to read, interpret, and act on Bash error messages.

By the end of this chapter, you'll be able to:
- Parse error message components
- Identify error types quickly
- Use debugging techniques effectively
- Build systematic troubleshooting habits

---

## 2.1 Anatomy of an Error Message

### Basic Error Structure

A typical Bash error message has these components:

```
script.sh: line 15: command: command not found
│         │        │        └─ Explanation
│         │        └─ Problem location (command name)
│         └─ Line number
└─ Source (script name or bash)
```

### Common Error Prefixes

```bash
bash:           # Error from bash itself
./script.sh:    # Error from your script
command:        # Error from external command
-bash:          # Error in interactive bash
```

---

## 2.2 Error Types and Categories

### Syntax Errors

**Pattern:** `syntax error near unexpected token`

```bash
# Example error
./script.sh: line 5: syntax error near unexpected token `fi'
```

**Common Causes:**
- Missing keywords (`then`, `do`, `done`, `fi`)
- Unmatched quotes or brackets
- Incorrect spacing

---

### Command Not Found Errors

**Pattern:** `command: command not found`

```bash
# Example error
bash: python3: command not found
```

**Common Causes:**
- Command not installed
- Not in PATH
- Typo in command name
- Missing execute permission

---

### Permission Errors

**Pattern:** `Permission denied`

```bash
# Example error
bash: ./script.sh: Permission denied
```

**Common Causes:**
- File not executable (`chmod +x` needed)
- Insufficient user permissions
- Directory permissions prevent access

---

### File/Directory Errors

**Pattern:** `No such file or directory`

```bash
# Example error
cat: file.txt: No such file or directory
```

**Common Causes:**
- File doesn't exist
- Wrong path (relative vs absolute)
- Typo in filename
- Hidden file (starts with .)

---

## 2.3 Reading Stack Traces

When scripts call other scripts, you get a "stack trace":

```bash
# Example stack trace
./main.sh: line 10: ./helper.sh: No such file or directory
```

**Reading it:**
1. Start at the innermost error (rightmost)
2. Work backward to find the root cause
3. Check each level for issues

---

## 2.4 Error Streams: stdout vs stderr

### Understanding Output Streams

```bash
# stdout (standard output) = File descriptor 1
echo "Normal output"

# stderr (standard error) = File descriptor 2
echo "Error message" >&2

# Both
command 2>&1  # Redirect stderr to stdout
```

### Redirecting Errors

```bash
# Discard errors
command 2>/dev/null

# Save errors to file
command 2>error.log

# Save both output and errors
command &>output.log

# Separate files
command >output.log 2>error.log
```

---

## 2.5 Debugging Techniques

### Using set -x (Execution Trace)

```bash
#!/bin/bash

# Enable execution trace
set -x

echo "This will show"
variable="test"
echo $variable

# Disable execution trace
set +x
```

**Output shows each command before execution:**
```
+ echo 'This will show'
This will show
+ variable=test
+ echo test
test
```

---

### Using set -e (Exit on Error)

```bash
#!/bin/bash

# Exit immediately if any command fails
set -e

echo "First command"
false  # This fails, script exits here
echo "This won't execute"
```

---

### Using set -u (Undefined Variables)

```bash
#!/bin/bash

# Treat unset variables as error
set -u

name="Alice"
echo $name      # Works
echo $undefined # Error: unbound variable
```

---

### Combining Debug Options

```bash
#!/bin/bash

# Strict mode: exit on error, undefined variables, pipe failures
set -euo pipefail

# Your script here
```

---

## 2.6 Common Error Messages Reference

### Quick Reference Table

| Error Message | Likely Cause | Quick Fix |
|--------------|--------------|-----------|
| `command not found` | Not in PATH or not installed | Check PATH, install command |
| `Permission denied` | No execute permission | `chmod +x file` |
| `No such file or directory` | File doesn't exist | Check path and filename |
| `syntax error` | Code syntax problem | Check brackets, quotes, keywords |
| `unexpected EOF` | Missing closing construct | Add `fi`, `done`, or `esac` |
| `unbound variable` | Variable not set | Initialize variable or use `${var:-}` |
| `bad substitution` | Wrong variable syntax | Check `${}` syntax |
| `binary file` | Trying to source binary | Use correct file type |

---

## 2.7 Hands-On: Error Message Detective

### Exercise Script

Save this as `error_detective.sh`:

```bash
#!/bin/bash

echo "Error Message Detective"
echo "======================="
echo

# Generate different errors
test_errors() {
    echo "1. Command not found error:"
    nonexistent_command 2>&1 || echo "   (Error demonstrated)"
    echo

    echo "2. Permission error:"
    touch /tmp/test_permission
    chmod 000 /tmp/test_permission
    cat /tmp/test_permission 2>&1 || echo "   (Error demonstrated)"
    rm /tmp/test_permission
    echo

    echo "3. File not found:"
    cat nonexistent_file.txt 2>&1 || echo "   (Error demonstrated)"
    echo

    echo "4. Syntax error in subscript:"
    bash -c 'if true then echo "missing semicolon"; fi' 2>&1 || echo "   (Error demonstrated)"
}

test_errors
```

Run it:
```bash
chmod +x error_detective.sh
./error_detective.sh
```

---

## 2.8 Diagnostic Tools

### Using type to Understand Commands

```bash
# Check what a command is
type ls        # Shows if alias, function, builtin, or external
type -a ls     # Shows all definitions
type -t ls     # Shows type only (alias/function/builtin/file)
```

### Using which and whereis

```bash
# Find command location
which python3

# Find all related files
whereis python3
```

### Using file to Check File Types

```bash
# Check file type
file script.sh
file /bin/bash
```

---

## 2.9 Building an Error Log

### Error Logging Script

```bash
#!/bin/bash

# Setup error logging
ERROR_LOG="/tmp/script_errors.log"

# Function to log errors
log_error() {
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] ERROR: $*" | tee -a "$ERROR_LOG" >&2
}

# Example usage
if ! some_command; then
    log_error "some_command failed on line $LINENO"
fi
```

---

## 2.10 Key Takeaways

✓ **Read error messages carefully** - Every part has meaning

✓ **Identify error type** - Syntax, command, permission, or file errors

✓ **Use debugging flags** - `set -x`, `set -e`, `set -u`

✓ **Check stderr separately** - Errors go to stderr, not stdout

✓ **Build systematic approach** - Always start with the error message

✓ **Keep a reference** - Common errors and solutions

---

## 2.11 Quick Reference Card

```bash
# Debugging flags
set -x          # Show commands before execution
set -e          # Exit on first error
set -u          # Error on undefined variables
set -o pipefail # Catch errors in pipes

# Check commands
type command    # What is this command?
which command   # Where is this command?
file filename   # What type of file?

# Redirect errors
2>/dev/null     # Discard errors
2>&1            # Combine stdout and stderr
&>file          # Save both to file
```

---

*End of Chapter 2*

**Next:** [Chapter 3: Command Not Found Errors](chapter-03-command-not-found.md)
