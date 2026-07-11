# 🌀 Defining Functions in Bash

There are two main forms (both work identically):

```bash
# Form 1
my_function() {
    echo "Hello from my_function"
}

# Form 2
function my_function {
    echo "Hello as well"
}
```

Call them just by writing their name:

```bash
my_function
```

---
# 🎯 Function Arguments

Inside a function:

- `$1` → the **first argument**
- `$2` → the **second argument**
- `$@` → all arguments as separate words
- `$#` → argument **count**
- `$*` → all arguments as a single string

## 🔍 Deep Dive into `$@` vs `$*`

- `$@` treats arguments separately. Perfect for looping:
    
    ```bash
    show_args() {
        for arg in "$@"; do
            echo "-> $arg"
        done
    }
    show_args "alpha beta" gamma
    ```
    
    - - Output:
        
        ```
        -> alpha beta
        -> gamma
        ```
        
- `$*` condenses them into one string if quoted:
    
    ```bash
    show_all() {
        echo "All at once: $*"
    }
    show_all "alpha beta" gamma
    ```
    
    Output:
    
    text
    
    ```
    All at once: alpha beta gamma
    ```
    
Subtle, but important.

---
# ⌨️ Scoped Variables — `local`

By default, variables inside a function **leak into the outside shell**. Hackers avoid polluting the “global namespace” by using `local`.

```bash
counter=100

increase() {
    local counter=1
    ((counter++))
    echo "Inside function: $counter"
}

increase
echo "Outside function: $counter"
```

Output:

text

```
Inside function: 2
Outside function: 100
```

---
# 🪄 Returning Values

Two paths here:

### 1. Return **status code** (0–255)

This is for success/failure, not actual data.

```bash
is_even() {
    if (( $1 % 2 == 0 )); then
        return 0   # success
    else
        return 1   # failure
    fi
}

if is_even 8; then
    echo "Yes, it's even!"
else
	echo "Nope, odd number."
fi
```

### 2. Return **real data via echo**

Bash doesn’t return strings directly; you `echo` them and capture with command substitution.

```bash
square() {
    echo $(( $1 * $1 ))
}

result=$(square 7)
echo "The square is $result"
```

---

# 🏗️ Functions Accepting Flexible Arguments

### Example 1 — Processing variable args

```bash
list_args() {
    echo "You passed $# args"
    index=1
    for arg in "$@"; do
        echo "Arg $index: $arg"
        ((index++))
    done
}
list_args red blue green
```

---
### Example 2 — Default values with parameter expansion

Bash

```
backup() {
    local src=${1:-"."}
    local dest=${2:-"/tmp/backup"}
    echo "Backing up $src to $dest"
}
backup "/etc"       # backs up /etc to /tmp/backup
backup               # uses . → /tmp/backup
```

---
### Example 3 — Handling “flags” like a mini CLI
```bash
process_flags() {
	while [[ $# -gt 0 ]]; do
		case $1 in
			-f|--file)
				file=$2
				shift 2
				;;
				-v|--verbose)
					verbose=true
					shift
					;;
				*)
					echo "Unknown arg: $1"
					shift
					;;
		esac
	done

    echo "File: $file"
    echo "Verbose: $verbose"
}
process_flags -f data.txt -v
```

---

# 🧰 More with `$@` and argument tricks

```bash
print_args() {
    echo "All args with \$*: $*"
    echo "All args with \$@: $@"
    echo "Loop through args: "
    for arg in "$@"; do
        echo "-> $arg"
    done
}
print_args alpha beta gamma
```

Notice subtlety:

- `$*` → all arguments **smushed into one string** (if quoted).
- `$@` → preserves each argument as separate items.

---
# 🤖 Advanced Function Weapons

### Default values (parameter expansion)

```bash
greet() {
    local name=${1:-"stranger"}
    echo "Hello, $name"
}
greet           # Hello, stranger
greet Neo       # Hello, Neo
```

### Named parameters (manual unpacking)

Because Bash doesn’t support named params, hackers do:

```bash
deploy() {
    local env=$1
    local service=$2
    echo "Deploying $service to $env"
}
deploy prod web
```

### Functions calling functions

Like cyber‑ninjas teaming up:

```bash
log() { echo "[LOG] $*"; }
action() { log "Performing action: $1"; }

action "Scan system"
```

---
# 🎮 Real Hacker‑ish Examples

**1. Batch renamer function**

```bash
rename_files() {
    prefix=$1
    count=1
    for f in *.*; do
        mv "$f" "${prefix}_${count}.${f##*.}"
        ((count++))
    done
}
rename_files hack
```

👉 All files in the folder renamed to `hack_1.jpg`, `hack_2.txt`, etc.

---
**2. Function library script**

```bash
say_time() {
    echo "Current time: $(date +%T)"
}
system_info() {
    echo "Uptime: $(uptime -p)"
    echo "Kernel: $(uname -r)"
}
```

Load these into your shell (`source functions.sh`) and boom—you’ve got a hacker toolkit at your fingertips.

---
### 3. Logging Utility

```bash
log() {
    local level=$1
    shift
    echo "[$(date +%T)] [$level] $*"
}
log INFO "System starting"
log ERROR "Disk space critical"
```
---
# 🕶️ Hacker’s Mindset Tip

Functions = modular thinking.  
Arguments = adaptability.  
Combine them, and you have **portable, combinable “moves”** you can call anytime. Reusability makes scripts short, sharp, and stealthy.

---
✨ Challenge for you:  
Write a function called `calc` that:

- Takes **three arguments**: two numbers and an operator (`+`, `-`, `*`, `/`).
- Performs the calculation.
- Prints the result.

For example:

```
calc 3 + 7   → 10
calc 20 / 5  → 4
```

