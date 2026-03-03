# 📜 Bash Scripting

> Frequently used Bash scripting patterns, constructs, and best practices.

[← Back to Home](../README.md)

---

## 📋 Table of Contents

- [Script Boilerplate](#script-boilerplate)
- [Variables](#variables)
- [String Operations](#string-operations)
- [Arithmetic](#arithmetic)
- [Arrays](#arrays)
- [Conditions & Tests](#conditions--tests)
- [Loops](#loops)
- [Functions](#functions)
- [Input & Arguments](#input--arguments)
- [File Operations](#file-operations)
- [Error Handling](#error-handling)
- [Process & Job Control](#process--job-control)
- [Here Documents & Strings](#here-documents--strings)
- [Common Patterns](#common-patterns)

---

## Script Boilerplate

```bash
#!/usr/bin/env bash
#
# Script: deploy.sh
# Description: Deploy application to Kubernetes
# Author: DevOps Team
# Version: 1.0.0
#

set -euo pipefail
# -e   exit on error
# -u   treat unset variables as errors
# -o pipefail   pipeline fails on first error

# Constants
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"

# Default values
ENV="${ENV:-staging}"
VERSION="${VERSION:-latest}"

# Logging helpers
log_info()  { echo "[INFO]  $(date +%H:%M:%S) $*"; }
log_warn()  { echo "[WARN]  $(date +%H:%M:%S) $*" >&2; }
log_error() { echo "[ERROR] $(date +%H:%M:%S) $*" >&2; }

# Cleanup on exit
cleanup() {
    log_info "Cleaning up..."
    # cleanup commands here
}
trap cleanup EXIT

main() {
    log_info "Starting deployment: env=$ENV, version=$VERSION"
    # main logic here
}

main "$@"
```

---

## Variables

```bash
# Assignment (no spaces around =)
name="Alice"
count=42
pi=3.14

# Read-only
readonly MAX_RETRIES=3

# Local variable (inside function)
local tmp_dir="/tmp/myapp"

# Command substitution
current_date=$(date +%Y-%m-%d)
current_date=`date +%Y-%m-%d`     # old style

# Default values
name="${1:-Alice}"                 # use "Alice" if $1 is unset or empty
name="${MYVAR:-default}"           # use default if MYVAR is unset
name="${MYVAR:=default}"           # assign default if unset

# String interpolation
echo "Hello, ${name}!"
echo "Count: $((count + 1))"

# Environment variables
export APP_ENV=production
unset APP_ENV

# Special variables
echo "$0"          # script name
echo "$1 $2"       # positional arguments
echo "$@"          # all arguments (array)
echo "$*"          # all arguments (string)
echo "$#"          # number of arguments
echo "$?"          # exit status of last command
echo "$$"          # PID of current shell
echo "$!"          # PID of last background process
echo "$LINENO"     # current line number
echo "$BASH_SOURCE"  # script path
```

---

## String Operations

```bash
name="Hello World"

# Length
echo "${#name}"              # 11

# Substring: ${var:offset:length}
echo "${name:0:5}"           # Hello
echo "${name:6}"             # World

# Remove prefix (shortest)
echo "${name#Hello }"        # World
# Remove prefix (longest)
echo "${name##*l}"           # d

# Remove suffix (shortest)
echo "${name%World}"         # Hello
# Remove suffix (longest)
echo "${name%%l*}"           # He

# Replace first occurrence
echo "${name/Hello/Hi}"      # Hi World
# Replace all occurrences
echo "${name//l/L}"          # HeLLo WorLd

# Uppercase / lowercase (bash 4+)
echo "${name,,}"             # hello world (lowercase)
echo "${name^^}"             # HELLO WORLD (uppercase)
echo "${name,}"              # hELLO WORLD (first char lower)
echo "${name^}"              # Hello World (first char upper)

# Check if contains substring
if [[ "$name" == *"World"* ]]; then echo "Contains World"; fi

# Trim whitespace
trimmed=$(echo "  hello  " | xargs)

# Split string
IFS=',' read -ra parts <<< "a,b,c"
for part in "${parts[@]}"; do echo "$part"; done
```

---

## Arithmetic

```bash
# Arithmetic expansion
a=10
b=3
echo $((a + b))     # 13
echo $((a - b))     # 7
echo $((a * b))     # 30
echo $((a / b))     # 3 (integer division)
echo $((a % b))     # 1 (modulo)
echo $((a ** b))    # 1000 (exponent)

# Increment
((count++))
((count--))
((count += 5))

# Float arithmetic (bash doesn't support floats natively)
result=$(echo "scale=2; 10 / 3" | bc)   # 3.33
result=$(python3 -c "print(10 / 3)")

# let command
let "a = 5 + 3"
let "b++"

# Comparison
if (( a > b )); then echo "a is greater"; fi
if (( a == 10 )); then echo "a is ten"; fi
```

---

## Arrays

```bash
# Declare array
fruits=("apple" "banana" "cherry")
declare -a nums=(1 2 3 4 5)

# Access elements
echo "${fruits[0]}"          # apple
echo "${fruits[1]}"          # banana
echo "${fruits[-1]}"         # cherry (last)

# All elements
echo "${fruits[@]}"          # apple banana cherry

# Array length
echo "${#fruits[@]}"         # 3

# Slice
echo "${fruits[@]:1:2}"      # banana cherry

# Append
fruits+=("date")
fruits[10]="elderberry"

# Remove element
unset fruits[1]

# Loop over array
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# Loop with index
for i in "${!fruits[@]}"; do
    echo "$i: ${fruits[$i]}"
done

# Associative array (hash map)
declare -A person
person[name]="Alice"
person[age]=30
person[city]="NYC"

echo "${person[name]}"       # Alice
echo "${!person[@]}"         # keys: name age city
echo "${person[@]}"          # values

for key in "${!person[@]}"; do
    echo "$key = ${person[$key]}"
done
```

---

## Conditions & Tests

```bash
# if-elif-else
if [ condition ]; then
    # commands
elif [ other ]; then
    # commands
else
    # commands
fi

# [[ ]] — modern test (preferred)
[[ -f file.txt ]]        # file exists and is regular file
[[ -d mydir ]]           # directory exists
[[ -e path ]]            # path exists
[[ -r file ]]            # file is readable
[[ -w file ]]            # file is writable
[[ -x file ]]            # file is executable
[[ -s file ]]            # file is not empty
[[ -L file ]]            # is symlink
[[ -z "$var" ]]          # string is empty
[[ -n "$var" ]]          # string is not empty
[[ "$a" == "$b" ]]       # string equality
[[ "$a" != "$b" ]]       # string inequality
[[ "$a" < "$b" ]]        # string less than
[[ "$str" =~ ^[0-9]+$ ]] # regex match

# Numeric comparison (use [[ ]] or (( )))
[[ "$a" -eq "$b" ]]      # equal
[[ "$a" -ne "$b" ]]      # not equal
[[ "$a" -lt "$b" ]]      # less than
[[ "$a" -le "$b" ]]      # less than or equal
[[ "$a" -gt "$b" ]]      # greater than
[[ "$a" -ge "$b" ]]      # greater than or equal

# Logical operators
[[ condition1 && condition2 ]]   # AND
[[ condition1 || condition2 ]]   # OR
[[ ! condition ]]                # NOT

# Examples
if [[ -f "config.yml" ]]; then
    echo "Config file found"
fi

if [[ -z "$API_KEY" ]]; then
    echo "ERROR: API_KEY not set" >&2
    exit 1
fi

if [[ "$ENV" == "production" && -n "$DB_URL" ]]; then
    deploy
fi
```

---

## Loops

```bash
# for loop
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

# C-style for loop
for (( i=0; i<5; i++ )); do
    echo "i=$i"
done

# Range
for i in {1..10}; do echo "$i"; done
for i in {0..20..5}; do echo "$i"; done    # step 5

# Loop over files
for file in *.txt; do
    echo "Processing: $file"
done

# Loop over command output
for host in $(cat hosts.txt); do
    ping -c 1 "$host"
done

# while loop
count=0
while [[ $count -lt 5 ]]; do
    echo "Count: $count"
    ((count++))
done

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt

# Infinite loop with break
while true; do
    if some_condition; then
        break
    fi
    sleep 5
done

# until loop
until [[ $status -eq 0 ]]; do
    check_status
    sleep 2
done

# Loop control
continue      # skip to next iteration
break         # exit loop
break 2       # exit 2 nested loops
```

---

## Functions

```bash
# Define a function
greet() {
    local name="$1"
    local greeting="${2:-Hello}"
    echo "$greeting, $name!"
}

# Call it
greet "Alice"
greet "Bob" "Hi"

# Return value (exit code only, 0-255)
is_even() {
    (( $1 % 2 == 0 ))
}

if is_even 4; then echo "Even"; fi

# Return a string via echo + command substitution
get_timestamp() {
    echo "$(date +%Y%m%d_%H%M%S)"
}
ts=$(get_timestamp)

# Function with default arguments
create_user() {
    local username="${1:?'username is required'}"
    local shell="${2:-/bin/bash}"
    local home="${3:-/home/$username}"
    useradd -m -s "$shell" -d "$home" "$username"
}

# Recursive function
factorial() {
    local n=$1
    (( n <= 1 )) && echo 1 || echo $(( n * $(factorial $((n-1))) ))
}
```

---

## Input & Arguments

```bash
# Command line arguments
script.sh arg1 arg2 arg3

# In script:
echo "Script: $0"
echo "First arg: $1"
echo "All args: $@"
echo "Arg count: $#"

# Shift arguments
shift          # removes $1, $2 becomes $1
shift 2        # removes $1 and $2

# getopts — option parsing
while getopts "e:v:h" opt; do
    case $opt in
        e) ENV="$OPTARG" ;;
        v) VERSION="$OPTARG" ;;
        h) show_help; exit 0 ;;
        ?) echo "Unknown option: -$OPTARG"; exit 1 ;;
    esac
done
shift $((OPTIND - 1))    # remove parsed options

# Usage: script.sh -e production -v 1.2.3

# Read user input
read -p "Enter your name: " name
read -sp "Enter password: " password    # silent (no echo)
read -t 10 -p "Continue? [y/N] " ans   # 10 second timeout

echo
if [[ "${ans,,}" == "y" ]]; then
    echo "Continuing..."
fi

# Select menu
select choice in "Deploy" "Rollback" "Status" "Quit"; do
    case $choice in
        Deploy)   deploy; break ;;
        Rollback) rollback; break ;;
        Status)   status; break ;;
        Quit)     exit 0 ;;
    esac
done
```

---

## File Operations

```bash
# Check if file/dir exists
[[ -f file.txt ]] && echo "file exists"
[[ -d mydir ]] || mkdir -p mydir

# Read file into variable
content=$(cat file.txt)

# Read file line by line
while IFS= read -r line; do
    process "$line"
done < input.txt

# Write to file
echo "content" > file.txt         # overwrite
echo "content" >> file.txt        # append

# Multiline write
cat > file.txt << 'EOF'
line 1
line 2
line 3
EOF

# Temp files
tmpfile=$(mktemp)
tmpdir=$(mktemp -d)
trap "rm -rf $tmpfile $tmpdir" EXIT

# Find and process files
find /var/log -name "*.log" -mtime +7 | while read -r file; do
    gzip "$file"
done

# File locking
exec 9>/tmp/mylock
if flock -n 9; then
    echo "Got lock, proceeding"
else
    echo "Another instance is running" >&2
    exit 1
fi
```

---

## Error Handling

```bash
# Exit on error
set -e

# Custom error handler
trap 'handle_error $LINENO' ERR
handle_error() {
    echo "ERROR at line $1" >&2
    exit 1
}

# Check command exit code
if ! command_that_might_fail; then
    echo "Command failed" >&2
    exit 1
fi

# OR operator for fallback
mkdir -p /opt/app || { echo "Cannot create dir"; exit 1; }

# Check required tools
require_tool() {
    command -v "$1" &>/dev/null || {
        echo "ERROR: $1 is required but not installed" >&2
        exit 1
    }
}

require_tool kubectl
require_tool docker
require_tool helm

# Retry function
retry() {
    local max_attempts="${1}"
    local delay="${2}"
    local command="${@:3}"
    local attempt=1

    until $command; do
        if (( attempt >= max_attempts )); then
            echo "Command failed after $attempt attempts: $command" >&2
            return 1
        fi
        echo "Attempt $attempt failed, retrying in ${delay}s..."
        sleep "$delay"
        ((attempt++))
    done
}

retry 3 5 kubectl get pods
```

---

## Process & Job Control

```bash
# Run in background
sleep 100 &
echo "PID: $!"

# Wait for background process
wait $!
wait            # wait for all background processes

# Run multiple processes in parallel
for host in web1 web2 web3; do
    ssh "$host" "sudo apt upgrade -y" &
done
wait
echo "All hosts upgraded"

# Check if process is running
if ps aux | grep -v grep | grep -q nginx; then
    echo "nginx is running"
fi

if pgrep nginx &>/dev/null; then
    echo "nginx is running"
fi

# Timeout a command
timeout 30 ./long-running-script.sh
if [[ $? -eq 124 ]]; then echo "Command timed out"; fi
```

---

## Here Documents & Strings

```bash
# Here document (heredoc)
cat << EOF
This is a
multiline string
with $variable expansion
EOF

# Heredoc without expansion
cat << 'EOF'
This is literal: $variable
EOF

# Heredoc to file
cat > /etc/nginx/conf.d/myapp.conf << EOF
server {
    listen 80;
    server_name ${DOMAIN};
    root /var/www/${APP_NAME};
}
EOF

# Heredoc to command
ssh user@host << 'EOF'
sudo systemctl restart nginx
echo "Nginx restarted"
EOF

# Here string
grep "pattern" <<< "$variable"
base64 -d <<< "SGVsbG8gV29ybGQ="
```

---

## Common Patterns

```bash
# Spinner for long operations
spinner() {
    local pid=$1
    local delay=0.1
    local spinstr='|/-\'
    while ps a | awk '{print $1}' | grep -q "$pid"; do
        local temp=${spinstr#?}
        printf " [%c] " "$spinstr"
        local spinstr=$temp${spinstr%"$temp"}
        sleep $delay
        printf "\b\b\b\b\b"
    done
}

# Progress bar
progress_bar() {
    local current=$1 total=$2
    local width=40
    local pct=$(( current * 100 / total ))
    local filled=$(( width * current / total ))
    local bar=$(printf "%${filled}s" | tr ' ' '#')
    printf "\r[%-${width}s] %d%%" "$bar" "$pct"
}

# Check if running as root
if [[ $EUID -ne 0 ]]; then
    echo "This script must be run as root" >&2
    exit 1
fi

# Or require root with sudo elevation
[[ $EUID -ne 0 ]] && exec sudo "$0" "$@"

# Colorized output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'    # No Color

echo -e "${GREEN}✓ Success${NC}"
echo -e "${RED}✗ Error${NC}"
echo -e "${YELLOW}⚠ Warning${NC}"

# Script directory
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# OS detection
case "$OSTYPE" in
    linux*)   OS="linux" ;;
    darwin*)  OS="macos" ;;
    cygwin*)  OS="windows" ;;
    *)        OS="unknown" ;;
esac

# Wait for service to be ready
wait_for_service() {
    local host="$1" port="$2" timeout="${3:-60}"
    local start=$(date +%s)
    while ! nc -z "$host" "$port" 2>/dev/null; do
        if (( $(date +%s) - start >= timeout )); then
            echo "Timed out waiting for $host:$port" >&2
            return 1
        fi
        sleep 1
    done
    echo "$host:$port is ready"
}

wait_for_service localhost 5432 60    # wait for postgres
```

---

[← Back to Home](../README.md)
