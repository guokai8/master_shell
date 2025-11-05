# Chapter 5: File and Directory Errors
## Mastering Path Resolution and Filesystem Navigation

---

## Overview

"No such file or directory" is one of the most common errors in Bash, yet it encompasses a wide variety of underlying issues. This chapter will teach you to diagnose and fix path-related errors, understand filesystem quirks, and handle tricky filenames on both MacOS and Linux.

By the end of this chapter, you'll master:
- Absolute vs. relative path resolution
- Handling spaces and special characters in filenames
- Symbolic and hard link troubleshooting
- Case sensitivity differences
- Hidden files and directories
- Globbing patterns and wildcards
- Platform-specific filesystem behaviors

---

## 5.1 Understanding "No Such File or Directory"

### The Basic Error

```bash
$ cat myfile.txt
cat: myfile.txt: No such file or directory
```

**What This Really Means:**
The shell cannot find the file you specified at the location you specified.

---

### Common Causes Checklist

```bash
#!/bin/bash
# file_not_found_diagnosis.sh

echo "File Not Found - Common Causes"
echo "==============================="
echo
echo "1. WRONG LOCATION"
echo "   You're in /home/user but file is in /home/user/documents"
echo
echo "2. TYPO IN NAME"
echo "   Looking for 'myfile.txt' but it's actually 'myfile.tx'"
echo
echo "3. CASE SENSITIVITY (Linux)"
echo "   Looking for 'File.txt' but it's actually 'file.txt'"
echo
echo "4. HIDDEN FILE"
echo "   File starts with dot: .myfile"
echo
echo "5. WHITESPACE IN NAME"
echo "   File is 'my file.txt' but you typed: cat my file.txt"
echo
echo "6. SPECIAL CHARACTERS"
echo "   File has characters like: file*.txt or file[1].txt"
echo
echo "7. SYMLINK BROKEN"
echo "   Symbolic link points to non-existent file"
echo
echo "8. DELETED/MOVED"
echo "   File was recently deleted or moved"
echo
echo "9. WRONG EXTENSION"
echo "   File is .TXT but you're looking for .txt"
echo
echo "10. PERMISSION ISSUES"
echo "    File exists but you can't see it"
```

---

## 5.2 Absolute vs. Relative Paths

### Understanding Path Types

```bash
#!/bin/bash
# path_tutorial.sh

echo "Path Types Tutorial"
echo "==================="
echo
echo "Current directory: $(pwd)"
echo

# Create test structure
mkdir -p /tmp/path_demo/subdir
cd /tmp/path_demo || exit
touch file1.txt subdir/file2.txt

echo "1. ABSOLUTE PATHS (start with /)"
echo "   Always start from root directory"
echo "   Example: $(pwd)/file1.txt"
echo "   Advantage: Works from anywhere"
echo

echo "2. RELATIVE PATHS (no leading /)"
echo "   Start from current directory"
echo "   Example: file1.txt or subdir/file2.txt"
echo "   Advantage: Portable, shorter"
echo

echo "3. HOME DIRECTORY (~)"
echo "   Expands to your home directory"
echo "   ~/Documents = $HOME/Documents"
echo

echo "4. PARENT DIRECTORY (..)"
echo "   Goes up one level"
echo "   ../file.txt = file in parent directory"
echo

echo "5. CURRENT DIRECTORY (.)"
echo "   Refers to current directory"
echo "   ./script.sh = run script in current dir"
echo

# Cleanup
cd /
rm -rf /tmp/path_demo
```

---

### Path Resolution Script

```bash
#!/bin/bash
# resolve_path.sh

resolve_and_check() {
    local path="$1"

    echo "Path: $path"
    echo "─────────────────────────────"

    # Check if absolute or relative
    if [[ "$path" == /* ]]; then
        echo "Type: Absolute path"
    else
        echo "Type: Relative path"
    fi

    # Show current directory
    echo "Current directory: $(pwd)"

    # Resolve to absolute
    if [[ -e "$path" ]]; then
        real_path=$(realpath "$path" 2>/dev/null || readlink -f "$path" 2>/dev/null)
        echo "Resolves to: $real_path"
        echo "✓ EXISTS"
    else
        echo "✗ DOES NOT EXIST"
        echo
        echo "Possible issues:"
        echo "  • Check spelling"
        echo "  • Verify you're in the right directory"
        echo "  • Use absolute path instead"
    fi
    echo
}

# Test with argument
if [[ -n "$1" ]]; then
    resolve_and_check "$1"
else
    echo "Usage: $0 <path>"
    echo
    echo "Examples:"
    echo "  $0 file.txt"
    echo "  $0 /etc/passwd"
    echo "  $0 ../other/file.txt"
fi
```

---

## 5.3 Handling Spaces and Special Characters

### The Space Problem

```bash
#!/bin/bash
# space_demo.sh

echo "Handling Spaces in Filenames"
echo "============================="
echo

# Create test file
touch "/tmp/my file.txt"
echo "test content" > "/tmp/my file.txt"

echo "Created file: 'my file.txt'"
echo

echo "PROBLEM: Unquoted filename"
echo "──────────────────────────"
echo '$ cat /tmp/my file.txt'
echo "Error: cat tries to open 'my' and 'file.txt' separately"
cat /tmp/my file.txt 2>&1 || true
echo

echo "SOLUTION 1: Double quotes"
echo "─────────────────────────"
echo '$ cat "/tmp/my file.txt"'
cat "/tmp/my file.txt"
echo

echo "SOLUTION 2: Single quotes"
echo "─────────────────────────"
echo "$ cat '/tmp/my file.txt'"
cat '/tmp/my file.txt'
echo

echo "SOLUTION 3: Escape spaces"
echo "─────────────────────────"
echo '$ cat /tmp/my\ file.txt'
cat /tmp/my\ file.txt
echo

# Cleanup
rm "/tmp/my file.txt"
```

---

### Special Characters Guide

```bash
#!/bin/bash
# special_chars.sh

echo "Special Characters in Filenames"
echo "================================"
echo

# Create test files
cd /tmp || exit
touch "file*.txt" "file[1].txt" 'file$var.txt' "file&test.txt"

echo "Created files with special characters"
echo

echo "PROBLEM CHARACTERS:"
echo "───────────────────"
echo

echo "1. Asterisk (*) - Wildcard"
echo "   File: 'file*.txt'"
echo "   Wrong: cat file*.txt  (expands to all matching)"
echo "   Right: cat 'file*.txt'"
cat 'file*.txt' 2>/dev/null || echo "   (empty file)"
echo

echo "2. Brackets [] - Character class"
echo "   File: 'file[1].txt'"
echo "   Wrong: cat file[1].txt  (glob expansion)"
echo "   Right: cat 'file[1].txt'"
cat 'file[1].txt' 2>/dev/null || echo "   (empty file)"
echo

echo "3. Dollar sign ($) - Variable expansion"
echo "   File: 'file\$var.txt'"
echo "   Wrong: cat file\$var.txt  (expands \$var)"
echo "   Right: cat 'file\$var.txt'"
cat 'file$var.txt' 2>/dev/null || echo "   (empty file)"
echo

echo "4. Ampersand (&) - Background process"
echo "   File: 'file&test.txt'"
echo "   Wrong: cat file&test.txt  (runs cat file in background)"
echo "   Right: cat 'file&test.txt'"
cat 'file&test.txt' 2>/dev/null || echo "   (empty file)"
echo

# Cleanup
rm 'file*.txt' 'file[1].txt' 'file$var.txt' 'file&test.txt'
```

---

### Safe Filename Handler

```bash
#!/bin/bash
# safe_filename.sh

# Function to handle any filename safely
safe_cat() {
    local file="$1"

    if [[ -z "$file" ]]; then
        echo "Usage: safe_cat <filename>"
        return 1
    fi

    if [[ ! -f "$file" ]]; then
        echo "Error: File not found: $file"
        return 1
    fi

    # Safely display contents
    cat -- "$file"
}

# Function to rename to safe filename
make_safe() {
    local original="$1"

    if [[ ! -e "$original" ]]; then
        echo "Error: File not found: $original"
        return 1
    fi

    # Replace problematic characters
    local safe=$(echo "$original" | tr ' &*?[]{}()<>|;`$!'"'"'\\' '_')

    if [[ "$original" == "$safe" ]]; then
        echo "✓ Filename is already safe: $original"
        return 0
    fi

    if [[ -e "$safe" ]]; then
        echo "Error: Target filename already exists: $safe"
        return 1
    fi

    mv -- "$original" "$safe"
    echo "✓ Renamed: '$original' → '$safe'"
}

# Menu
case "${1:-}" in
    cat)
        safe_cat "$2"
        ;;
    fix)
        make_safe "$2"
        ;;
    *)
        echo "Safe Filename Handler"
        echo "====================="
        echo
        echo "Usage:"
        echo "  $0 cat <file>    - Safely display file"
        echo "  $0 fix <file>    - Rename to safe filename"
        ;;
esac
```

---

## 5.4 Symbolic Links and Hard Links

### Understanding Links

```bash
#!/bin/bash
# links_tutorial.sh

echo "Symbolic Links vs Hard Links"
echo "============================="
echo

cd /tmp || exit
echo "Original" > original.txt

echo "1. SYMBOLIC LINKS (Soft Links)"
echo "───────────────────────────────"
ln -s original.txt symlink.txt
echo "Created: symlink.txt → original.txt"
echo "Content: $(cat symlink.txt)"
ls -l symlink.txt
echo

echo "2. HARD LINKS"
echo "─────────────"
ln original.txt hardlink.txt
echo "Created: hardlink.txt (hard link to original.txt)"
echo "Content: $(cat hardlink.txt)"
ls -l hardlink.txt original.txt
echo

echo "3. WHAT HAPPENS WHEN ORIGINAL IS DELETED"
echo "─────────────────────────────────────────"
rm original.txt

echo "Symbolic link (broken):"
cat symlink.txt 2>&1 || echo "  ✗ Symlink broken"
ls -l symlink.txt

echo
echo "Hard link (still works):"
cat hardlink.txt
echo "  ✓ Hard link still works"
echo

# Cleanup
rm symlink.txt hardlink.txt
```

---

### Fixing Broken Symlinks

```bash
#!/bin/bash
# fix_broken_symlinks.sh

find_broken_symlinks() {
    local search_dir="${1:-.}"
    
    echo "Finding broken symlinks in: $search_dir"
    echo "═══════════════════════════════════════"
    echo
    
    local found=0
    
    while IFS= read -r -d '' link; do
        if [[ ! -e "$link" ]]; then
            ((found++))
            echo "BROKEN: $link"
            echo "  Target: $(readlink "$link")"
            echo "  Location: $(dirname "$link")"
            echo
        fi
    done < <(find "$search_dir" -type l -print0 2>/dev/null)
    
    if ((found == 0)); then
        echo "✓ No broken symlinks found"
    else
        echo "Found $found broken symlink(s)"
    fi
}

fix_symlink() {
    local link="$1"
    local new_target="$2"
    
    if [[ ! -L "$link" ]]; then
        echo "Error: $link is not a symbolic link"
        return 1
    fi
    
    local old_target=$(readlink "$link")
    
    echo "Fixing symlink: $link"
    echo "  Old target: $old_target"
    echo "  New target: $new_target"
    
    if [[ ! -e "$new_target" ]]; then
        echo "Warning: New target doesn't exist"
        read -p "Continue anyway? (y/N) " -r
        if [[ ! $REPLY =~ ^[Yy]$ ]]; then
            return 1
        fi
    fi
    
    rm "$link"
    ln -s "$new_target" "$link"
    echo "✓ Fixed"
}

case "${1:-}" in
    find)
        find_broken_symlinks "$2"
        ;;
    fix)
        fix_symlink "$2" "$3"
        ;;
    *)
        echo "Broken Symlink Fixer"
        echo "===================="
        echo
        echo "Usage:"
        echo "  $0 find [directory]           - Find broken symlinks"
        echo "  $0 fix <symlink> <new_target> - Fix a symlink"
        ;;
esac
```

---

## 5.5 Case Sensitivity Issues

### MacOS vs Linux

```bash
#!/bin/bash
# case_sensitivity.sh

echo "Case Sensitivity Test"
echo "===================="
echo

cd /tmp || exit

# Create test file
echo "content" > testfile.txt

echo "Created: testfile.txt"
echo

echo "Testing case sensitivity..."
echo

# Test different cases
test_cases=("testfile.txt" "TestFile.txt" "TESTFILE.TXT" "testFile.TXT")

for case_test in "${test_cases[@]}"; do
    echo "Testing: $case_test"
    if [[ -f "$case_test" ]]; then
        echo "  ✓ Found"
    else
        echo "  ✗ Not found"
    fi
done

echo
if [[ -f "TestFile.txt" ]]; then
    echo "✓ Filesystem is case-INSENSITIVE (likely macOS)"
    echo "  Different cases refer to the same file"
else
    echo "✓ Filesystem is case-SENSITIVE (likely Linux)"
    echo "  Different cases are different files"
fi

# Cleanup
rm testfile.txt
```

---

## 5.6 Hidden Files and Directories

### Working with Hidden Files

```bash
#!/bin/bash
# hidden_files.sh

list_hidden() {
    local dir="${1:-.}"
    
    echo "Hidden files in: $dir"
    echo "══════════════════════════════════════════════════════════"
    echo
    
    local count=$(find "$dir" -maxdepth 1 -name ".*" ! -name "." ! -name ".." 2>/dev/null | wc -l)
    
    if ((count == 0)); then
        echo "No hidden files found"
        return 0
    fi
    
    echo "Found $count hidden item(s):"
    echo
    
    # Files
    echo "FILES:"
    find "$dir" -maxdepth 1 -name ".*" -type f ! -name "." ! -name ".." 2>/dev/null | while read -r file; do
        echo "  $(basename "$file")"
        ls -lh "$file"
    done
    echo
    
    # Directories
    echo "DIRECTORIES:"
    find "$dir" -maxdepth 1 -name ".*" -type d ! -name "." ! -name ".." 2>/dev/null | while read -r dir_path; do
        echo "  $(basename "$dir_path")/"
        ls -lhd "$dir_path"
    done
}

hide_file() {
    local file="$1"
    
    if [[ ! -e "$file" ]]; then
        echo "Error: File not found"
        return 1
    fi
    
    local dir=$(dirname "$file")
    local base=$(basename "$file")
    
    if [[ "$base" == .* ]]; then
        echo "File is already hidden"
        return 0
    fi
    
    local hidden="$dir/.$base"
    
    if [[ -e "$hidden" ]]; then
        echo "Error: Hidden version already exists"
        return 1
    fi
    
    mv "$file" "$hidden"
    echo "✓ Hidden: $file → $hidden"
}

unhide_file() {
    local file="$1"
    
    if [[ ! -e "$file" ]]; then
        echo "Error: File not found"
        return 1
    fi
    
    local dir=$(dirname "$file")
    local base=$(basename "$file")
    
    if [[ "$base" != .* ]]; then
        echo "File is not hidden"
        return 0
    fi
    
    local visible="$dir/${base#.}"
    
    if [[ -e "$visible" ]]; then
        echo "Error: Visible version already exists"
        return 1
    fi
    
    mv "$file" "$visible"
    echo "✓ Unhidden: $file → $visible"
}

case "${1:-}" in
    list)
        list_hidden "$2"
        ;;
    hide)
        hide_file "$2"
        ;;
    unhide)
        unhide_file "$2"
        ;;
    *)
        echo "Hidden File Operations"
        echo "======================"
        echo
        echo "Usage:"
        echo "  $0 list [dir]    - List hidden files"
        echo "  $0 hide <file>   - Make file hidden"
        echo "  $0 unhide <file> - Make file visible"
        ;;
esac
```

---

## 5.7 File Globbing and Wildcards

### Glob Patterns Guide

```bash
#!/bin/bash
# glob_tutorial.sh

echo "File Globbing Tutorial"
echo "======================"
echo

cd /tmp || exit
mkdir glob_demo
cd glob_demo || exit

# Create test files
touch file1.txt file2.txt file10.txt
touch data1.csv data2.csv
touch test_a.sh test_b.sh
touch "file with spaces.txt"

echo "Created test files"
ls
echo

echo "1. ASTERISK (*) - Match any characters"
echo "───────────────────────────────────────"
echo "$ ls *.txt"
ls *.txt
echo

echo "2. QUESTION MARK (?) - Match single character"
echo "──────────────────────────────────────────────"
echo "$ ls file?.txt"
ls file?.txt
echo

echo "3. BRACKETS [] - Character class"
echo "─────────────────────────────────"
echo "$ ls file[12].txt"
ls file[12].txt
echo

echo "$ ls data[1-2].csv"
ls data[1-2].csv
echo

echo "4. BRACES {} - Multiple patterns"
echo "─────────────────────────────────"
echo "$ ls *.{txt,csv}"
echo "(Lists all .txt and .csv files)"
ls *.txt *.csv
echo

echo "5. NEGATION [!]"
echo "───────────────"
echo "$ ls file[!1].txt"
ls file[!1].txt
echo

# Cleanup
cd /tmp
rm -rf glob_demo
```

---

### Safe Globbing Practices

```bash
#!/bin/bash
# safe_globbing.sh

echo "Safe Globbing Practices"
echo "======================="
echo

# Enable nullglob (empty array if no matches)
shopt -s nullglob

echo "1. ALWAYS QUOTE ARRAY EXPANSIONS"
echo "─────────────────────────────────"
files=("file 1.txt" "file 2.txt")
echo "Array: (\"file 1.txt\" \"file 2.txt\")"
echo
echo "Wrong: for f in \${files[@]}; do"
for f in ${files[@]}; do
    echo "  Item: '$f'"  # Splits on spaces!
done
echo
echo "Correct: for f in \"\${files[@]}\"; do"
for f in "${files[@]}"; do
    echo "  Item: '$f'"  # Preserves spaces
done
echo

echo "2. CHECK FOR MATCHES"
echo "────────────────────"
pattern="*.nonexistent"
files=($pattern)

if ((${#files[@]} == 0)); then
    echo "No files match: $pattern"
else
    echo "Found ${#files[@]} file(s)"
fi
echo

echo "3. USE GLOBS, NOT \$(ls)"
echo "────────────────────────"
echo "Wrong: for f in \$(ls *.txt); do"
echo "Right: for f in *.txt; do"
echo

shopt -u nullglob
```

---

## 5.8 Ultimate File Finder

```bash
#!/bin/bash
# ultimate_file_finder.sh

find_file() {
    local filename="$1"
    local start_dir="${2:-.}"

    if [[ -z "$filename" ]]; then
        echo "Usage: $0 <filename> [start_directory]"
        return 1
    fi

    echo "╔══════════════════════════════════════════════════════════╗"
    echo "║         ULTIMATE FILE FINDER                             ║"
    echo "╚══════════════════════════════════════════════════════════╝"
    echo
    echo "Searching for: $filename"
    echo "Starting from: $start_dir"
    echo "═══════════════════════════════════════════════════════════"
    echo

    # 1. Exact match
    echo "1. EXACT NAME MATCH"
    echo "───────────────────────────────────────────────────────────"
    local exact=$(find "$start_dir" -name "$filename" -type f 2>/dev/null | head -5)
    if [[ -n "$exact" ]]; then
        echo "✓ Found:"
        echo "$exact"
    else
        echo "✗ No exact matches"
    fi
    echo

    # 2. Case-insensitive
    echo "2. CASE-INSENSITIVE"
    echo "───────────────────────────────────────────────────────────"
    local icase=$(find "$start_dir" -iname "$filename" -type f 2>/dev/null | head -5)
    if [[ -n "$icase" ]]; then
        echo "✓ Found:"
        echo "$icase"
    else
        echo "✗ No matches"
    fi
    echo

    # 3. Partial match
    echo "3. PARTIAL MATCH"
    echo "───────────────────────────────────────────────────────────"
    local partial=$(find "$start_dir" -iname "*$filename*" -type f 2>/dev/null | head -5)
    if [[ -n "$partial" ]]; then
        echo "✓ Found (showing first 5):"
        echo "$partial"
    else
        echo "✗ No matches"
    fi
    echo

    # 4. Common locations
    echo "4. CHECKING COMMON LOCATIONS"
    echo "───────────────────────────────────────────────────────────"
    local common_dirs=(
        "$HOME/Documents"
        "$HOME/Downloads"
        "$HOME/Desktop"
        "/tmp"
    )

    for dir in "${common_dirs[@]}"; do
        if [[ -d "$dir" ]]; then
            if find "$dir" -maxdepth 2 -iname "*$filename*" -type f 2>/dev/null | grep -q .; then
                echo "✓ Found in: $dir"
            fi
        fi
    done
    echo

    echo "═══════════════════════════════════════════════════════════"
    echo "SEARCH COMPLETE"
}

find_file "$@"
```

---

## 5.9 Key Takeaways

✓ **Understand paths** - Absolute vs. relative

✓ **Quote filenames** - Protect spaces and special characters

✓ **Know symlinks** - They can break if targets move

✓ **Remember case** - MacOS usually insensitive, Linux sensitive

✓ **Hidden files** - Start with dot, use ls -a

✓ **Quote globs** - Variables yes, glob patterns no

✓ **Test on both** - MacOS and Linux differences matter

---

## 5.10 Quick Reference

```bash
# Path operations
pwd                     # Current directory
realpath file          # Absolute path
dirname /path/to/file  # Directory part
basename /path/to/file # Filename part

# Handling special names
cat "file with spaces.txt"
rm -- -filename        # File starting with dash

# Finding files
find . -name "file.txt"     # Exact
find . -iname "file.txt"    # Case-insensitive
find . -name "*.txt"        # Pattern

# Symlinks
ln -s target link      # Create symlink
readlink link          # Show target
find . -xtype l        # Find broken symlinks

# Hidden files
ls -a                  # Show hidden
find . -name ".*"      # Find hidden

# Globs
*.txt                  # All .txt
file?.txt              # file + one char
file[123].txt          # file1, 2, or 3
```

---

*End of Chapter 5*

**Next:** [Chapter 6: Syntax Errors](chapter-06-syntax-errors.md)
