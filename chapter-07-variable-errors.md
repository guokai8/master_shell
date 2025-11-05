# Chapter 7: Variable and Parameter Errors
## Mastering Variables, Parameters, and Expansions

---

## Overview

Variables are the foundation of shell scripting, yet they're a common source of errors. From unbound variables to parameter expansion issues, this chapter will teach you to handle variables correctly and avoid common pitfalls on both MacOS and Linux.

By the end of this chapter, you'll master:
- Variable declaration and assignment
- Dealing with unbound variable errors
- Parameter expansion techniques
- Array handling and iteration
- Environment vs. shell variables
- Variable scope and subshells
- Special variables and their uses
- Default values and error handling

---

## 7.1 Understanding Variable Basics

### Variable Declaration and Assignment

```bash
#!/bin/bash
# variable_basics.sh

echo "Bash Variable Basics"
echo "===================="
echo

cat << 'EOF'
SYNTAX RULES:
─────────────

✓ CORRECT:
  var="value"           # No spaces around =
  var='value'           # Single quotes (literal)
  var="value $other"    # Double quotes (expansion)
  var=$(command)        # Command substitution
  var=$((5 + 3))        # Arithmetic

✗ WRONG:
  var = "value"         # Spaces around =
  $var="value"          # $ on left side
  var= "value"          # Space before value

ACCESSING:
──────────

$var                    # Simple access
${var}                  # Explicit (safer)
"$var"                  # Quoted (recommended)

NAMING:
───────

✓ Valid: var, VAR, _var, var1, my_var
✗ Invalid: 1var, my-var, my var
EOF
```

---

## 7.2 Unbound Variable Errors

### Understanding "Unbound Variable"

```bash
#!/bin/bash
# unbound_variables.sh

echo "Unbound Variable Errors"
echo "======================="
echo

cat << 'EOF'
WHAT IS IT:
───────────
A variable that has never been set

WITHOUT set -u:
───────────────
• Expands to empty string
• Silent bugs
• Hard to debug

WITH set -u:
────────────
• Script exits on unbound variable
• Catches typos early
• Recommended for production

ERROR MESSAGE:
──────────────
bash: VARIABLE_NAME: unbound variable
EOF

echo
echo "DEMONSTRATION:"
echo "──────────────"
echo

# Without set -u
echo "1. Without set -u:"
echo "   \$undefined_var = '$undefined_var' (empty)"
echo

# With set -u
echo "2. With set -u:"
(
    set -u
    echo "   Trying \$undefined_var..."
    echo "   Value: $undefined_var" 2>&1 || echo "   ✗ Error caught!"
)
```

---

### Handling Unbound Variables

```bash
#!/bin/bash
# handling_unbound.sh

echo "Handling Unbound Variables"
echo "=========================="
echo

cat << 'EOF'
TECHNIQUES:
───────────

1. DEFAULT VALUES:
   ${var:-default}        # Use default if unset
   ${var:=default}        # Assign default if unset
   ${var:?error}          # Exit with error if unset
   ${var:+alternative}    # Use alternative if set

2. CHECK BEFORE USE:
   if [[ -v var ]]; then
       echo "$var"
   fi

3. INITIALIZE:
   var="${var:-}"         # Empty if unset

4. USE set -u:
   set -u
   name="${name:-Anonymous}"
EOF

echo
echo "EXAMPLES:"
echo "─────────"
echo

# Example 1: Default value
unset username
echo "1. Default value (:-)"
echo "   username unset"
echo "   \${username:-Guest} = ${username:-Guest}"
echo "   username still unset"
echo

# Example 2: Assign default
unset username
echo "2. Assign default (:=)"
echo "   \${username:=Guest} = ${username:=Guest}"
echo "   username now = '$username'"
echo

# Example 3: Error if unset
echo "3. Error if unset (:?)"
unset required_var
result=$(: ${required_var:?Variable required} 2>&1) || echo "   Error: $result"
```

---

## 7.3 Parameter Expansion

### Basic Parameter Expansion

```bash
#!/bin/bash
# parameter_expansion.sh

echo "Parameter Expansion Guide"
echo "========================="
echo

cat << 'EOF'
BASIC:
──────
${var}                  # Basic expansion
${var:-default}         # Default if unset
${var:=default}         # Assign default
${var:?error}           # Error if unset
${var:+value}           # Use if set

STRING:
───────
${#var}                 # Length
${var:offset}           # Substring from offset
${var:offset:length}    # Substring with length
${var#pattern}          # Remove prefix (shortest)
${var##pattern}         # Remove prefix (longest)
${var%pattern}          # Remove suffix (shortest)
${var%%pattern}         # Remove suffix (longest)

SUBSTITUTION:
─────────────
${var/old/new}          # Replace first
${var//old/new}         # Replace all
${var/#old/new}         # Replace at start
${var/%old/new}         # Replace at end

CASE:
─────
${var^^}                # Uppercase all
${var,,}                # Lowercase all
${var^}                 # Uppercase first
${var,}                 # Lowercase first
EOF

echo
echo "EXAMPLES:"
echo "─────────"
echo

filename="document.txt.backup"
echo "filename=\"$filename\""
echo
echo "\${#filename} = ${#filename}  (length)"
echo "\${filename#*.} = ${filename#*.}  (remove first ext)"
echo "\${filename##*.} = ${filename##*.}  (get last ext)"
echo "\${filename%.backup} = ${filename%.backup}  (remove suffix)"
echo "\${filename%%.*} = ${filename%%.*}  (remove all ext)"
```

---

### Advanced Examples

```bash
#!/bin/bash
# advanced_expansion.sh

echo "Advanced Parameter Expansion"
echo "============================"
echo

# Extract filename parts
fullpath="/home/user/documents/report.2024.pdf"
echo "Path: $fullpath"
echo
echo "Basename: ${fullpath##*/}"
echo "Directory: ${fullpath%/*}"
echo "Extension: ${fullpath##*.}"
echo "Without ext: ${fullpath%.*}"
echo

# Convert paths
windows="C:\\Users\\John\\Documents"
echo "Windows: $windows"
echo "Unix: ${windows//\\//}"
echo

# Remove prefix/suffix
url="https://www.example.com/page.html"
echo "URL: $url"
echo "Without protocol: ${url#*://}"
domain="${url#*://}"
echo "Domain only: ${domain%%/*}"
```

---

## 7.4 Arrays

### Array Basics

```bash
#!/bin/bash
# arrays.sh

echo "Bash Arrays Guide"
echo "================="
echo

cat << 'EOF'
DECLARATION:
────────────

arr=()                  # Empty array
arr=("a" "b" "c")       # With values
arr[0]="first"          # Indexed assignment
mapfile -t arr < file   # From file

ACCESS:
───────

${arr[0]}               # First element
${arr[-1]}              # Last element (Bash 4.3+)
${arr[@]}               # All elements
"${arr[@]}"             # All (quoted)
${#arr[@]}              # Count
${!arr[@]}              # All indices

OPERATIONS:
───────────

arr+=(element)          # Append
unset arr[1]            # Remove element
arr=()                  # Clear

ITERATION:
──────────

for item in "${arr[@]}"; do
    echo "$item"
done
EOF

echo
echo "EXAMPLES:"
echo "─────────"
echo

colors=("red" "green" "blue")
echo "Array: (${colors[@]})"
echo "Count: ${#colors[@]}"
echo "First: ${colors[0]}"
echo "Last: ${colors[-1]}"
echo

echo "Iteration:"
for color in "${colors[@]}"; do
    echo "  $color"
done
```

---

### Common Array Mistakes

```bash
#!/bin/bash
# array_mistakes.sh

echo "Common Array Mistakes"
echo "====================="
echo

# Mistake 1: Not quoting
echo "MISTAKE 1: Not quoting expansion"
files=("file 1.txt" "file 2.txt")
echo "Array: (\"file 1.txt\" \"file 2.txt\")"
echo
echo "Wrong: for f in \${files[@]}"
for f in ${files[@]}; do
    echo "  '$f'"  # Splits!
done
echo
echo "Right: for f in \"\${files[@]}\""
for f in "${files[@]}"; do
    echo "  '$f'"  # Preserved
done
echo

# Mistake 2: Counting wrong
echo "MISTAKE 2: Wrong count syntax"
arr=("a" "b" "c")
echo "Wrong: \${#arr} = ${#arr}  (length of first)"
echo "Right: \${#arr[@]} = ${#arr[@]}  (count)"
```

---

## 7.5 Environment Variables

### Understanding Environment Variables

```bash
#!/bin/bash
# environment_variables.sh

echo "Environment Variables"
echo "====================="
echo

cat << 'EOF'
SHELL vs ENVIRONMENT:
─────────────────────

SHELL VARIABLE:
• Local to current shell
• Not inherited
• Set: var="value"

ENVIRONMENT VARIABLE:
• Inherited by children
• Exported
• Set: export VAR="value"

COMMON ENV VARS:
────────────────
PATH, HOME, USER, SHELL,
PWD, HOSTNAME, LANG, TERM

OPERATIONS:
───────────
export VAR="value"   # Create/export
echo "$VAR"          # Access
unset VAR            # Remove
env                  # List all
printenv VAR         # Print specific
EOF

echo
echo "DEMONSTRATION:"
echo "──────────────"
echo

shell_var="local"
export env_var="exported"

echo "Set shell_var=\"local\""
echo "Set export env_var=\"exported\""
echo
echo "In subshell:"
bash -c 'echo "  shell_var=$shell_var (empty)"'
bash -c 'echo "  env_var=$env_var (visible)"'
```

---

## 7.6 Variable Scope and Subshells

### Understanding Scope

```bash
#!/bin/bash
# variable_scope.sh

echo "Variable Scope"
echo "=============="
echo

cat << 'EOF'
SCOPE RULES:
────────────

GLOBAL:
• Default scope
• Available everywhere
• Available in functions

LOCAL:
• Declared with 'local'
• Function-only
• Shadows globals

SUBSHELL:
• Changes don't affect parent
• Created by: ( ), $( ), |
• Gets copy of variables
EOF

echo
echo "EXAMPLES:"
echo "─────────"
echo

# Global variable
global_var="I am global"
echo "1. Global: $global_var"

show_global() {
    echo "   In function: $global_var"
}
show_global
echo

# Local shadows global
counter=10
echo "2. Global counter=$counter"

increment() {
    local counter=0
    ((counter++))
    echo "   Function (local): counter=$counter"
}
increment
echo "   After function: counter=$counter (unchanged)"
echo

# Subshell doesn't affect parent
value="original"
echo "3. Original: $value"
(
    value="changed"
    echo "   Subshell: $value"
)
echo "   After subshell: $value (unchanged)"
```

---

### Pipeline Subshell Issue

```bash
#!/bin/bash
# pipeline_subshell.sh

echo "Pipeline Subshell Problem"
echo "========================="
echo

# Problem: Variable in pipeline
sum=0
echo "Initial sum: $sum"
echo

echo "Wrong (pipeline):"
echo "1 2 3" | while read n; do
    ((sum += n))
done
echo "After pipeline: sum=$sum (still 0!)"
echo

# Solution: Process substitution
sum=0
echo "Right (process substitution):"
while read n; do
    ((sum += n))
done < <(echo "1 2 3")
echo "After loop: sum=$sum (correct!)"
```

---

## 7.7 Special Variables

### Built-in Special Variables

```bash
#!/bin/bash
# special_variables.sh

echo "Special Variables"
echo "================="
echo

cat << 'EOF'
POSITIONAL:
───────────
$0          Script name
$1-$9       Arguments
$#          Argument count
$@          All arguments (separate)
$*          All arguments (single)

PROCESS:
────────
$$          Current PID
$!          Last background PID
$?          Last exit status
$-          Shell options

SPECIAL:
────────
$_          Last argument
$RANDOM     Random number
$LINENO     Line number
$HOSTNAME   Hostname
EOF

echo
echo "CURRENT VALUES:"
echo "───────────────"
echo "\$0 = $0"
echo "\$# = $#"
echo "\$$ = $$"
echo "\$RANDOM = $RANDOM"
echo "\$LINENO = $LINENO"
echo

# Exit status
true
echo "After 'true': \$? = $?"
false
echo "After 'false': \$? = $?"
```

---

## 7.8 Variable Debugging Tools

### Variable Inspector

```bash
#!/bin/bash
# variable_inspector.sh

inspect_variable() {
    local var_name="$1"
    
    if [[ -z "$var_name" ]]; then
        echo "Usage: $0 <variable_name>"
        return 1
    fi
    
    echo "Variable Inspector: $var_name"
    echo "═══════════════════════════════"
    echo
    
    # Check if variable is set
    if [[ -v "$var_name" ]]; then
        echo "✓ Variable is set"
        
        # Get value
        local value="${!var_name}"
        echo "Value: '$value'"
        echo "Length: ${#value}"
        echo "Type: $(declare -p "$var_name" 2>/dev/null | cut -d' ' -f2 || echo "regular")"
        
        # Check if exported
        if [[ "${!var_name@a}" == *x* ]]; then
            echo "✓ Exported (environment variable)"
        else
            echo "• Shell variable only"
        fi
        
        # Check if readonly
        if [[ "${!var_name@a}" == *r* ]]; then
            echo "✓ Read-only"
        fi
        
    else
        echo "✗ Variable is not set"
        echo
        echo "Suggestions:"
        echo "  • Check spelling"
        echo "  • Initialize: $var_name=\"\""
        echo "  • Use default: \${$var_name:-default}"
    fi
}

inspect_variable "$1"
```

---

## 7.9 Key Takeaways

✓ **Always quote variables** - "$var" not $var

✓ **Use set -u** - Catch unbound variables

✓ **Provide defaults** - ${var:-default}

✓ **Use local in functions** - Prevent global modification

✓ **Know subshell limits** - Variables don't persist

✓ **Quote array expansion** - "${array[@]}"

✓ **Export when needed** - Only for child processes

✓ **Understand special vars** - $?, $!, $@

---

## 7.10 Quick Reference

```bash
# Declaration
var="value"
readonly VAR="constant"
declare -i num=42

# Access
$var
${var}
"$var"

# Defaults
${var:-default}        # Use default
${var:=default}        # Assign default
${var:?error}          # Error if unset

# String operations
${#var}                # Length
${var:start:len}       # Substring
${var#pattern}         # Remove prefix
${var%pattern}         # Remove suffix
${var/old/new}         # Replace
${var^^}               # Uppercase

# Arrays
arr=(a b c)
${arr[0]}
${arr[@]}
${#arr[@]}

# Environment
export VAR="value"
unset VAR

# Special variables
$0 $1 $2 ...           # Script and arguments
$# $@ $*               # Argument info
$$ $! $?               # Process info
```

---

*End of Chapter 7*

**Next:** [Chapter 8: Process and Job Control Errors](chapter-08-process-errors.md)
