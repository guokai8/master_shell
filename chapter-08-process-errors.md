# Chapter 8: Process and Job Control Errors
## Managing Processes, Jobs, and Signal Handling

---

## Overview

Process management and job control are essential Bash skills. This chapter teaches you to manage processes, handle signals properly, and debug issues with background jobs, exit codes, and process communication.

By the end of this chapter, you'll master:
- Process IDs and parent/child relationships
- Foreground and background job control
- Exit codes and error propagation
- Signal handling with trap
- Killing and managing stuck processes
- Process substitution techniques
- Preventing zombie processes
- Concurrent script execution

---

## 8.1 Understanding Processes

### Process Basics

```bash
#!/bin/bash
# process_basics.sh

echo "Understanding Processes"
echo "======================"
echo

cat << 'EOF'
CONCEPTS:
─────────

PROCESS ID (PID):
• Unique identifier
• Shown in ps, top, $$
• Used to manage process

PARENT PID (PPID):
• Creator's PID
• Forms process tree

STATES:
───────
R   Running
S   Sleeping
D   Uninterruptible sleep
T   Stopped
Z   Zombie

COMMANDS:
─────────
ps              Current processes
ps aux          All processes
pstree          Process tree
top             Monitor
EOF

echo
echo "CURRENT PROCESS:"
echo "────────────────"
echo "PID: $$"
echo "PPID: $PPID"
echo "User: $(whoami)"
echo "Dir: $(pwd)"
```

---

## 8.2 Job Control

### Foreground and Background

```bash
#!/bin/bash
# job_control.sh

echo "Job Control"
echo "==========="
echo

cat << 'EOF'
BASICS:
───────

FOREGROUND:
• Blocks shell
• Can interrupt (Ctrl+C)

BACKGROUND:
• Runs independently
• Started with &
• Shell remains interactive

COMMANDS:
─────────

command &           Run in background
jobs                List jobs
fg %1               Foreground job 1
bg %1               Background job 1
kill %1             Kill job 1
wait                Wait for all
wait PID            Wait for specific

SHORTCUTS:
──────────

Ctrl+C              Kill (SIGINT)
Ctrl+Z              Stop (SIGTSTP)
EOF

echo
echo "DEMONSTRATION:"
echo "──────────────"
echo

# Background job
sleep 3 &
pid=$!
echo "Started sleep 3 in background (PID: $pid)"
jobs
wait $pid
echo "Job completed"
```

---

## 8.3 Exit Codes

### Understanding Exit Codes

```bash
#!/bin/bash
# exit_codes.sh

echo "Exit Codes"
echo "=========="
echo

cat << 'EOF'
BASICS:
───────

• 0 = Success
• 1-255 = Failure
• Stored in $?

COMMON CODES:
─────────────

0       Success
1       General error
2       Misuse of builtin
126     Not executable
127     Command not found
128+N   Fatal signal N
130     Ctrl+C (SIGINT)

USAGE:
──────

exit 0              Success
exit 1              Error
return 0            Function success

CHECKING:
─────────

if command; then
    echo "Success"
fi

command
if [ $? -eq 0 ]; then
    echo "Success"
fi
EOF

echo
echo "EXAMPLES:"
echo "─────────"
echo

true
echo "After 'true': \$? = $?"

false
echo "After 'false': \$? = $?"

nonexistent 2>/dev/null || ec=$?
echo "After nonexistent: \$? = $ec"
```

---

### Exit Code Best Practices

```bash
#!/bin/bash
# exit_code_practices.sh

set -euo pipefail

echo "Exit Code Best Practices"
echo "========================"
echo

# Define constants
readonly EXIT_SUCCESS=0
readonly EXIT_ERROR=1
readonly EXIT_CONFIG_ERROR=2

echo "1. Use named constants"
echo "   readonly EXIT_SUCCESS=0"
echo

# Check important commands
echo "2. Check important commands"
if cp /etc/passwd /tmp/test_$$ 2>/dev/null; then
    echo "   ✓ Copy succeeded"
    rm /tmp/test_$$
else
    echo "   ✗ Copy failed"
fi
echo

# Propagate errors in pipelines
echo "3. Use pipefail"
set -o pipefail
if echo "test" | grep test | wc -l >/dev/null; then
    echo "   ✓ Pipeline succeeded"
fi

if false | echo "continues" 2>/dev/null; then
    echo "   Pipeline succeeded"
else
    echo "   ✗ Pipeline failed (caught)"
fi
```

---

## 8.4 Signal Handling with Trap

### Understanding Signals

```bash
#!/bin/bash
# signals.sh

echo "Signals"
echo "======="
echo

cat << 'EOF'
COMMON SIGNALS:
───────────────

SIGINT (2)      Ctrl+C
SIGTERM (15)    Termination
SIGKILL (9)     Force kill
SIGHUP (1)      Hangup
SIGTSTP (20)    Ctrl+Z

TRAP SYNTAX:
────────────

trap 'commands' SIGNAL
trap 'commands' EXIT
trap 'commands' ERR

PSEUDO-SIGNALS:
───────────────

EXIT            Script exits
ERR             Command fails
DEBUG           Before command
RETURN          Function returns

EXAMPLES:
─────────

trap 'cleanup' EXIT
trap 'echo Interrupted' INT
trap 'handle_error $LINENO' ERR
EOF
```

---

### Practical Trap Examples

```bash
#!/bin/bash
# trap_examples.sh

set -euo pipefail

echo "Trap Examples"
echo "============="
echo

# Example 1: Cleanup
echo "1. Cleanup on exit"
temp_dir=$(mktemp -d)

cleanup() {
    echo "   Cleaning up: $temp_dir"
    rm -rf "$temp_dir"
}

trap cleanup EXIT

echo "   Created: $temp_dir"
touch "$temp_dir/test.txt"
echo "   (Will be cleaned up)"
echo

# Example 2: Graceful shutdown
echo "2. Graceful shutdown"

shutdown_handler() {
    echo "   Shutting down gracefully..."
}

trap shutdown_handler INT TERM

echo "   Trap set for INT and TERM"
echo

# Example 3: Error handler
echo "3. Error handling"

error_handler() {
    echo "   Error on line $1"
}

trap 'error_handler $LINENO' ERR

echo "   Trap set for ERR"
```

---

## 8.5 Killing Processes

### Kill Command

```bash
#!/bin/bash
# killing_processes.sh

echo "Killing Processes"
echo "================="
echo

cat << 'EOF'
SYNTAX:
───────

kill PID            TERM signal
kill -SIGNAL PID    Specific signal
kill -9 PID         Force kill
kill -15 PID        Graceful term

killall NAME        By name
pkill NAME          By pattern

CHECKING:
─────────

kill -0 PID         Check if running

STRATEGY:
─────────

1. Try graceful:
   kill -TERM $PID
   sleep 2

2. Check still running:
   if kill -0 $PID 2>/dev/null; then
       kill -KILL $PID
   fi
EOF

echo
echo "DEMONSTRATION:"
echo "──────────────"
echo

# Start process
sleep 100 &
pid=$!
echo "Started process: $pid"

# Graceful kill
echo "Sending TERM..."
kill -TERM $pid
wait $pid 2>/dev/null || true
echo "✓ Process terminated"
```

---

### Safe Termination

```bash
#!/bin/bash
# safe_termination.sh

kill_gracefully() {
    local pid=$1
    local timeout=${2:-10}

    echo "Killing $pid gracefully..."

    if ! kill -0 $pid 2>/dev/null; then
        echo "  Process doesn't exist"
        return 1
    fi

    # Try TERM
    echo "  Sending TERM..."
    kill -TERM $pid 2>/dev/null || return 1

    # Wait
    local count=0
    while kill -0 $pid 2>/dev/null && ((count < timeout)); do
        sleep 1
        ((count++))
    done

    # Check
    if ! kill -0 $pid 2>/dev/null; then
        echo "  ✓ Terminated gracefully"
        return 0
    fi

    # Force
    echo "  Timeout, sending KILL..."
    kill -9 $pid 2>/dev/null || true
    sleep 1

    if ! kill -0 $pid 2>/dev/null; then
        echo "  ✓ Killed"
    else
        echo "  ✗ Failed to kill"
        return 1
    fi
}

# Demo
sleep 100 &
pid=$!
echo "Started: $pid"
kill_gracefully $pid 2
```

---

## 8.6 Zombie and Orphan Processes

### Understanding Zombies

```bash
#!/bin/bash
# zombies_orphans.sh

cat << 'EOF'
ZOMBIE PROCESS:
───────────────

DEFINITION:
• Process completed
• Exit status not read
• Shows as <defunct>
• Takes process table entry

CAUSE:
• Parent doesn't call wait()

FIX:
• Parent must wait
• Or kill parent (init reaps)

ORPHAN PROCESS:
───────────────

DEFINITION:
• Parent has died
• Adopted by init (PID 1)

NOT A PROBLEM:
• Automatically cleaned up

PREVENTION:
───────────

For Zombies:
1. Always wait
2. Use trap to wait on EXIT
3. Handle SIGCHLD

trap 'wait' EXIT
EOF
```

---

## 8.7 Process Substitution

### Understanding Process Substitution

```bash
#!/bin/bash
# process_substitution.sh

echo "Process Substitution"
echo "===================="
echo

cat << 'EOF'
SYNTAX:
───────

<(command)          Output as file
>(command)          Input as file

ADVANTAGES:
───────────

• Avoid subshell issues
• Use output as filename
• Compare outputs
• Parallel processing

EXAMPLES:
─────────

1. Compare:
   diff <(ls dir1) <(ls dir2)

2. Preserve variables:
   while read line; do
       ((count++))
   done < <(cat file)

3. Multiple inputs:
   paste <(seq 1 5) <(seq 10 15)
EOF

echo
echo "DEMONSTRATION:"
echo "──────────────"
echo

# Variable persistence
echo "Variable in loop:"
count=0
while read word; do
    ((count++))
done < <(echo "one two three")
echo "Count: $count (preserved!)"
```

---

## 8.8 Concurrent Execution

### Running Tasks Concurrently

```bash
#!/bin/bash
# concurrent_execution.sh

echo "Concurrent Execution"
echo "===================="
echo

# Simple concurrent
echo "1. Simple concurrent"
task1() { echo "  Task 1"; sleep 1; }
task2() { echo "  Task 2"; sleep 1; }

task1 &
task2 &

echo "  Waiting..."
wait
echo "  ✓ Complete"
echo

# Limited parallelism
echo "2. Limited parallelism"
max_jobs=3

wait_for_slot() {
    while (($(jobs -r | wc -l) >= max_jobs)); do
        sleep 0.1
    done
}

echo "  Processing 10 items (max $max_jobs concurrent)"
for i in {1..10}; do
    wait_for_slot
    (echo "  Item $i"; sleep 0.5) &
done

wait
echo "  ✓ All complete"
```

---

## 8.9 Process Debugging Tools

### Process Monitor

```bash
#!/bin/bash
# process_monitor.sh

monitor_process() {
    local pid="$1"
    local interval="${2:-1}"
    
    if [[ -z "$pid" ]]; then
        echo "Usage: $0 <PID> [interval]"
        return 1
    fi
    
    echo "Monitoring process $pid (interval: ${interval}s)"
    echo "Press Ctrl+C to stop"
    echo
    
    while kill -0 "$pid" 2>/dev/null; do
        echo "$(date): PID $pid"
        ps -p "$pid" -o pid,ppid,state,pcpu,pmem,etime,cmd 2>/dev/null || {
            echo "Process $pid no longer exists"
            break
        }
        echo
        sleep "$interval"
    done
}

list_children() {
    local pid="$1"
    
    if [[ -z "$pid" ]]; then
        echo "Usage: list_children <PID>"
        return 1
    fi
    
    echo "Children of process $pid:"
    pgrep -P "$pid" 2>/dev/null | while read child; do
        echo "  $child: $(ps -p $child -o cmd --no-headers 2>/dev/null)"
    done
}

case "${1:-}" in
    monitor)
        monitor_process "$2" "$3"
        ;;
    children)
        list_children "$2"
        ;;
    *)
        echo "Process Tools"
        echo "============="
        echo
        echo "Usage:"
        echo "  $0 monitor <PID> [interval]  - Monitor process"
        echo "  $0 children <PID>            - List child processes"
        ;;
esac
```

---

## 8.10 Key Takeaways

✓ **Save background PIDs** - Store $! for management

✓ **Wait for children** - Prevent zombies

✓ **Check exit codes** - Use $? immediately

✓ **Use trap for cleanup** - Always trap EXIT

✓ **Kill gracefully** - TERM before KILL

✓ **Handle signals** - Trap INT and TERM

✓ **Use process substitution** - Avoid pipelines

✓ **Limit concurrency** - Don't spawn unlimited jobs

---

## 8.11 Quick Reference

```bash
# Process info
$$                      # Current PID
$!                      # Last background PID
ps aux                  # List all

# Job control
command &               # Background
jobs                    # List jobs
wait                    # Wait for all

# Killing
kill $pid               # TERM
kill -9 $pid            # KILL
kill -0 $pid            # Check running

# Signals
trap 'cmd' EXIT         # On exit
trap 'cmd' INT          # On Ctrl+C

# Exit codes
exit 0                  # Success
$?                      # Last exit

# Process substitution
<(command)              # Output as file
diff <(cmd1) <(cmd2)    # Compare
```

---

*End of Chapter 8*

**Next:** [Conclusion](#conclusion)

---

# CONCLUSION

**Congratulations!** You've completed **"The Complete Guide to Bash Errors: From Beginner to Expert"**.

## What You've Learned:

1. **Chapter 1**: Installation and environment setup
2. **Chapter 2**: Understanding error messages
3. **Chapter 3**: Command not found errors
4. **Chapter 4**: Permission and access errors
5. **Chapter 5**: File and directory errors
6. **Chapter 6**: Syntax errors
7. **Chapter 7**: Variable and parameter errors
8. **Chapter 8**: Process and job control errors

## Next Steps:

✓ Practice with the exercises
✓ Create your own scripts
✓ Build a reference library
✓ Contribute to the Bash community
✓ Keep learning!

## Resources:

- Bash Manual: `man bash`
- ShellCheck: https://www.shellcheck.net/
- Bash Guide: https://mywiki.wooledge.org/BashGuide

**Thank you for reading!** Every error is a learning opportunity. Happy scripting! 🚀
