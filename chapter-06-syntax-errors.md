# Chapter 6: Syntax Errors
## Understanding and Fixing Bash Script Syntax Issues

---

## Overview

Syntax errors are among the most common—and often most cryptic—errors in Bash scripting. Unlike logical errors that produce wrong results, syntax errors prevent your script from running at all. This chapter will teach you to identify, understand, and fix every type of Bash syntax error.

By the end of this chapter, you'll master:
- Reading and interpreting syntax error messages
- Quoting rules and escaping special characters
- Bracket and brace matching
- Line continuation and heredoc syntax
- Common syntax pitfalls and how to avoid them
- Using syntax checkers and linters
- Debugging complex syntax issues

---

## 6.1 Understanding Syntax Error Messages

### The Anatomy of Syntax Errors

```bash
#!/bin/bash
# syntax_error_example.sh

# Intentional error
if [ $x -eq 5 ]
then
    echo "x is 5"
# Missing fi
```

**Error Output:**
```
./script.sh: line 8: syntax error: unexpected end of file
```

**Components:**
- `./script.sh` - The script file
- `line 8` - Where Bash gave up (not always the actual error!)
- `syntax error` - Type of error
- `unexpected end of file` - What Bash found

---

### Common Syntax Error Patterns

```bash
#!/bin/bash
# common_syntax_errors.sh

cat << 'EOF'
COMMON SYNTAX ERRORS:
─────────────────────

1. "syntax error near unexpected token"
   Cause: Unmatched quotes, brackets, or braces

2. "unexpected end of file"
   Cause: Missing fi, done, or esac

3. "unexpected EOF while looking for matching"
   Cause: Unclosed quote or here-document

4. "syntax error: operand expected"
   Cause: Bad arithmetic expression

5. "syntax error: invalid arithmetic operator"
   Cause: Wrong operator in (( )) or let

6. "command not found"
   Sometimes: Actually a syntax error (missing quotes)

7. "bad substitution"
   Cause: Wrong variable syntax ${var}
EOF
```

---

## 6.2 Quoting and Escaping

### The Three Types of Quotes

```bash
#!/bin/bash
# quoting_guide.sh

echo "Quoting in Bash"
echo "==============="
echo

name="Alice"
price=10

# Single quotes - Literal
echo "1. SINGLE QUOTES (' ')"
echo '   $name = literal $name'
echo '   Price: $10 (no expansion)'
echo

# Double quotes - Expansion
echo '2. DOUBLE QUOTES (" ")'
echo "   \$name = $name (expanded)"
echo "   Price: \$$price (escaped $)"
echo

# No quotes - Word splitting
echo "3. NO QUOTES"
text="one two three"
echo "   With quotes:"
for word in "$text"; do
    echo "     $word"
done
echo "   Without quotes:"
for word in $text; do
    echo "     $word"
done
echo

# ANSI-C quoting
echo '4. ANSI-C QUOTING ($'\''...'\'')'
echo $'   Line 1\nLine 2'
echo $'   Tab:\there'
```

---

### Quoting Problems and Solutions

```bash
#!/bin/bash
# quoting_problems.sh

echo "Common Quoting Problems"
echo "======================="
echo

# Problem 1: Unmatched quotes
echo "PROBLEM 1: Unmatched quote"
echo 'Code: echo "Hello'
echo "Error: unexpected EOF while looking for matching \`\"'"
echo 'Fix: echo "Hello"'
echo

# Problem 2: Variable in single quotes
echo "PROBLEM 2: Variable won't expand"
name="Bob"
echo 'Code: echo '\''Hello $name'\'''
echo 'Hello $name'
echo 'Fix: echo "Hello $name"'
echo "Hello $name"
echo

# Problem 3: Filename with spaces
echo "PROBLEM 3: Spaces in filenames"
echo 'Wrong: cat my file.txt'
echo 'Right: cat "my file.txt"'
echo

# Problem 4: Special characters
echo "PROBLEM 4: Dollar sign literal"
echo 'Wrong: echo "Price is $10"'
echo "Wrong result: Price is $10" #$1 expands!
echo 'Right: echo "Price is \$10"'
echo "Right result: Price is \$10"
```

---

## 6.3 Bracket and Parenthesis Matching

### Understanding Bash Brackets

```bash
#!/bin/bash
# brackets_guide.sh

cat << 'EOF'
BASH BRACKETS GUIDE:
════════════════════

1. SINGLE BRACKETS [ ]
   • Test command
   • POSIX compatible
   • Requires spaces
   • Quote variables
   Example: if [ "$x" = "5" ]; then

2. DOUBLE BRACKETS [[ ]]
   • Bash extended test
   • Pattern matching
   • Safer with variables
   Example: if [[ $x == 5 ]]; then

3. SINGLE PARENTHESES ( )
   • Subshell execution
   • Command grouping
   Example: (cd /tmp && ls)

4. DOUBLE PARENTHESES (( ))
   • Arithmetic evaluation
   • C-style syntax
   Example: ((x = 5 + 3))

5. CURLY BRACES { }
   • Variable expansion
   • Command grouping
   Example: ${var:-default}

6. ANGLE BRACKETS < >
   • Redirection
   • Here-documents
   Example: cmd < input.txt
EOF
```

---

### Bracket Matching Checker

```bash
#!/bin/bash
# check_brackets.sh

check_brackets() {
    local file="$1"

    if [[ ! -f "$file" ]]; then
        echo "Usage: $0 <script_file>"
        return 1
    fi

    echo "Checking Brackets in: $file"
    echo "════════════════════════════"
    echo

    # Count brackets
    local single_open=$(grep -o '\[' "$file" | wc -l)
    local single_close=$(grep -o '\]' "$file" | wc -l)
    local paren_open=$(grep -o '(' "$file" | wc -l)
    local paren_close=$(grep -o ')' "$file" | wc -l)
    local brace_open=$(grep -o '{' "$file" | wc -l)
    local brace_close=$(grep -o '}' "$file" | wc -l)

    echo "Bracket Count:"
    echo "  [ : $single_open"
    echo "  ] : $single_close"
    echo "  ( : $paren_open"
    echo "  ) : $paren_close"
    echo "  { : $brace_open"
    echo "  } : $brace_close"
    echo

    local issues=0

    if ((single_open != single_close)); then
        echo "✗ Single brackets unbalanced"
        ((issues++))
    fi

    if ((paren_open != paren_close)); then
        echo "✗ Parentheses unbalanced"
        ((issues++))
    fi

    if ((brace_open != brace_close)); then
        echo "✗ Braces unbalanced"
        ((issues++))
    fi

    if ((issues == 0)); then
        echo "✓ All brackets balanced"
    fi
}

check_brackets "$1"
```

---

## 6.4 Control Structure Syntax

### If/Then/Else/Fi

```bash
#!/bin/bash
# if_syntax.sh

cat << 'EOF'
IF STATEMENT SYNTAX:
════════════════════

1. Simple if:
   if condition; then
       commands
   fi

2. If-else:
   if condition; then
       commands
   else
       commands
   fi

3. If-elif-else:
   if condition1; then
       commands
   elif condition2; then
       commands
   else
       commands
   fi

COMMON ERRORS:
══════════════

✗ Missing then:
  if [ $x -eq 5 ]
      echo "five"
  fi

✗ Missing fi:
  if [ $x -eq 5 ]; then
      echo "five"

✓ CORRECT:
  if [ $x -eq 5 ]; then
      echo "five"
  fi
EOF
```

---

### Loop Syntax

```bash
#!/bin/bash
# loop_syntax.sh

cat << 'EOF'
FOR LOOP SYNTAX:
════════════════

1. C-style:
   for ((i=0; i<10; i++)); do
       commands
   done

2. For-in:
   for item in list; do
       commands
   done

WHILE LOOP:
═══════════

while condition; do
    commands
done

UNTIL LOOP:
═══════════

until condition; do
    commands
done

COMMON ERRORS:
══════════════

✗ Missing do:
  for i in 1 2 3
      echo $i
  done

✗ Missing done:
  for i in 1 2 3; do
      echo $i

✓ CORRECT:
  for i in 1 2 3; do
      echo $i
  done
EOF
```

---

### Case Statement Syntax

```bash
#!/bin/bash
# case_syntax.sh

cat << 'EOF'
CASE SYNTAX:
════════════

case $variable in
    pattern1)
        commands
        ;;
    pattern2|pattern3)
        commands
        ;;
    *)
        default
        ;;
esac

PATTERNS:
═════════
• Exact: "yes")
• Wildcard: *) or yes*)
• Multiple: pattern1|pattern2)
• Range: [Yy]es)

COMMON ERRORS:
══════════════

✗ Missing ;;
  case $x in
      1) echo "one"
      2) echo "two"
  esac

✗ Missing esac:
  case $x in
      1) echo "one" ;;

✓ CORRECT:
  case $x in
      1) echo "one" ;;
      2) echo "two" ;;
      *) echo "other" ;;
  esac
EOF
```

---

## 6.5 Here-Documents

### Here-Document Syntax

```bash
#!/bin/bash
# heredoc_syntax.sh

cat << 'EOF'
HERE-DOCUMENT SYNTAX:
═════════════════════

BASIC:
command << DELIMITER
content
DELIMITER

RULES:
1. Opening << DELIMITER on same line
2. Closing DELIMITER alone on its line
3. Closing DELIMITER at start (no spaces!)

VARIATIONS:
───────────

1. With expansion:
   cat << EOF
   Hello $name
   EOF

2. Without expansion (quoted):
   cat << 'EOF'
   Hello $name
   EOF

3. Strip leading tabs (<<-):
   cat <<- EOF
       Indented
   EOF

COMMON ERRORS:
══════════════

✗ Indented closing:
  cat << EOF
  text
      EOF  # Error!

✗ Space after closing:
  cat << EOF
  text
  EOF   # Space here - Error!

✓ CORRECT:
  cat << EOF
  text
  EOF
EOF
```

---

## 6.6 Using Syntax Checkers

### bash -n (Dry Run)

```bash
#!/bin/bash
# check_syntax.sh

check_script() {
    local script="$1"

    if [[ ! -f "$script" ]]; then
        echo "Usage: $0 <script>"
        return 1
    fi

    echo "Checking syntax: $script"
    echo "════════════════════════"
    echo

    # Check syntax without running
    if bash -n "$script" 2>&1; then
        echo "✓ No syntax errors"
    else
        echo "✗ Syntax errors found"
        return 1
    fi
    echo

    # Check shebang
    if head -1 "$script" | grep -q '^#!'; then
        echo "✓ Shebang present: $(head -1 "$script")"
    else
        echo "⚠ No shebang"
    fi
    echo

    # Check executable
    if [[ -x "$script" ]]; then
        echo "✓ Executable"
    else
        echo "⚠ Not executable (chmod +x $script)"
    fi
}

check_script "$1"
```

---

### ShellCheck Integration

```bash
#!/bin/bash
# shellcheck_wrapper.sh

run_shellcheck() {
    local script="$1"

    if ! command -v shellcheck &>/dev/null; then
        echo "ShellCheck not installed"
        echo "Install: brew install shellcheck (macOS)"
        echo "         sudo apt install shellcheck (Linux)"
        return 1
    fi

    echo "Running ShellCheck on: $script"
    echo "══════════════════════════════"
    echo

    shellcheck "$script"

    local exit_code=$?

    if ((exit_code == 0)); then
        echo
        echo "✓ No issues found"
    fi

    return $exit_code
}

run_shellcheck "$1"
```

---

## 6.7 Advanced Syntax Issues

### Function Syntax

```bash
#!/bin/bash
# function_syntax.sh

cat << 'EOF'
FUNCTION SYNTAX:
════════════════

Method 1:
function name() {
    commands
}

Method 2:
name() {
    commands
}

COMMON ERRORS:
══════════════

✗ Missing () with function keyword:
  function name {
      commands
  }

✗ Missing braces:
  name()
      commands

✗ Space before ():
  name () {
      commands
  }

✓ CORRECT:
  name() {
      commands
  }

BEST PRACTICES:
═══════════════
• Use method 2 (more portable)
• Always declare local variables
• Return meaningful exit codes
EOF
```

---

### Arithmetic Syntax

```bash
#!/bin/bash
# arithmetic_syntax.sh

echo "Arithmetic in Bash"
echo "=================="
echo

echo "1. ARITHMETIC EXPANSION \$((...))"
echo "─────────────────────────────────"
x=5
y=3
echo "x = $x, y = $y"
echo "x + y = $((x + y))"
echo "x * y = $((x * y))"
echo

echo "2. ARITHMETIC EVALUATION ((...)))"
echo "─────────────────────────────────"
((result = x + y * 2))
echo "result = $result"
echo

echo "3. COMMON ERRORS"
echo "────────────────"
echo "✗ Space in variable: \$( ( x + y ) )"
echo "✗ String in arithmetic: \$(( \"5\" + \"3\" ))"
echo "✗ Missing \$ for variables in some contexts"
echo
echo "✓ CORRECT:"
echo "  \$((x + y))"
echo "  ((x = 5))"
echo "  let \"x = 5\""
```

---

## 6.8 Debugging Syntax Issues

### Line-by-Line Checker

```bash
#!/bin/bash
# syntax_debugger.sh

debug_syntax() {
    local script="$1"
    
    if [[ ! -f "$script" ]]; then
        echo "Usage: $0 <script>"
        return 1
    fi
    
    echo "Debugging syntax: $script"
    echo "═════════════════════════"
    echo
    
    local line_num=1
    
    # Create temporary file for progressive checking
    local temp_file=$(mktemp)
    
    while IFS= read -r line; do
        echo "$line" >> "$temp_file"
        
        # Check syntax up to this line
        if ! bash -n "$temp_file" 2>/dev/null; then
            echo "✗ Syntax error at or before line $line_num:"
            echo "   $line"
            echo
            bash -n "$temp_file"
            break
        fi
        
        ((line_num++))
    done < "$script"
    
    rm -f "$temp_file"
}

debug_syntax "$1"
```

---

## 6.9 Key Takeaways

✓ **Read error messages carefully** - Line number is a clue, not always exact

✓ **Check constructs** - Every if needs fi, every do needs done

✓ **Quote variables** - Always use "$var" not $var

✓ **Match brackets** - Use a syntax-aware editor

✓ **Use bash -n** - Check syntax before running

✓ **Install ShellCheck** - Best tool for catching errors

✓ **Mind spaces** - Critical in [ ], =, and function definitions

---

## 6.10 Quick Reference

```bash
# Syntax checking
bash -n script.sh          # Check syntax
bash -x script.sh          # Trace execution
shellcheck script.sh       # Static analysis

# If statement
if condition; then
    commands
fi

# Loop
for item in list; do
    commands
done

# Case
case $var in
    pattern)
        commands
        ;;
esac

# Function
name() {
    commands
}

# Here-document
cat << EOF
content
EOF

# Arithmetic
$((expression))          # Expansion
((expression))           # Evaluation

# Quoting
"$var"                   # Expand variables
'$var'                   # Literal
$'string\n'              # ANSI-C quoting
```

---

*End of Chapter 6*

**Next:** [Chapter 7: Variable and Parameter Errors](chapter-07-variable-errors.md)
