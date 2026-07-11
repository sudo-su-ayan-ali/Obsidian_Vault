## 🟢 Anatomy of an If Statement

In bash, an `if` statement looks like this:

```bash
if [ condition ]
then
    # commands when true
else
    # commands when false
fi
```

- `if` → keyword to start.
- `[ condition ]` → must be **true** or **false**.
- `then` → block starts here.
- `fi` → ends the if statement (“if” backwards; cute, right?).

---
## 🟡 Testing Conditions

The magic happens inside those brackets `[ ]`. Here’s what you can test:

### 1. String Conditions

```bash
if [ "$user" == "neo" ]; then
    echo "Welcome, The One."
fi

if [ "$string" != "hello" ]; then
    echo "It’s not hello..."
fi
```

⚠️ Always wrap variables in quotes (`"$var"`) to avoid errors if they’re empty or have spaces.  
(`$name == ""` would break, `"$name" == ""` is safe.)

---
### 2. Numeric Conditions

Bash **doesn’t** use `<`, `>` inside `[ ]` for numbers. You use `-eq`, `-gt`, `-lt`, etc.

- `-eq` = equal to
- `-ne` = not equal
- `-gt` = greater than
- `-lt` = less than
- `-ge` = greater or equal
- `-le` = less or equal

Example:

```bash
x=7
if [ $x -gt 5 ]; then
    echo "$x is greater than 5"
else
    echo "$x is not greater than 5"
fi
```

---
### 3. File Conditions

Hackers love files 👾. Common file tests:

- `-e file` : exists
- `-f file` : exists and is a file
- `-d dir` : exists and is a directory
- `-s file` : file exists and is not empty
- `-r/-w/-x file` : readable / writable / executable

Example:

```bash
if [ -f "secret.txt" ]; then
    echo "File exists!"
else
    echo "No secrets here..."
fi
```

---
## 🟠 Combining Conditions

Use **logical operators**:

- `&&` → AND (all must be true)
- `||` → OR (at least one true)
- `!` → NOT

```bash
if [ $x -gt 5 ] && [ $x -lt 10 ]; then
    echo "$x is between 5 and 10"
fi
```

Or in double-brackets (preferred style):


```bash
if [[ $x -gt 5 && $x -lt 10 ]]; then
    echo "$x is between 5 and 10"
fi
```

---
## 🔵 Else If (elif)

Sometimes choices are not binary, so use `elif`:


```bash
read -p "Enter your level [1-3]: " lvl

if [ "$lvl" -eq 1 ]; then
    echo "Rookie Hacker 👶"
elif [ "$lvl" -eq 2 ]; then
    echo "Intermediate Hacker 🕶️"
elif [ "$lvl" -eq 3 ]; then
    echo "Elite Hacker 👑"
else
    echo "Invalid level!"
fi
```

---
## 🟣 Nested Ifs

Like inception: an if inside another if.

```bash
if [ $USER == "root" ]; then
    if [ -f "/etc/passwd" ]; then
        echo "You have root AND passwd file exists. Dangerous combo. 😈"
    fi
else
    echo "Not root, relax..."
fi
```

---
## ⚡ Pro Tips

- Use `[[ ... ]]` instead of `[ ... ]` → it’s more flexible (can handle `==` with patterns, doesn’t complain as much).
- Always **quote variables** in conditions: `"$var"`.
- For readability, put keywords on their own lines (just like the examples, don’t cram everything).
- Test your logic step by step before going full hacker-mode.

---
### 🏆 Quick Example Script


```bash
#!/bin/bash
read -p "Enter a number: " n

if [[ $n -lt 0 ]]; then
    echo "Negative number."
elif [[ $n -eq 0 ]]; then
    echo "Zero."
else
    echo "Positive number."
fi
```

---



