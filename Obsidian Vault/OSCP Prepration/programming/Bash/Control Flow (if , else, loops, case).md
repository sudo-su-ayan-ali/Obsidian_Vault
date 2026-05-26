## 1️⃣ If / Else: Decisions

This is the “choose your adventure” part.

Bash

```bash
if COMMAND1; then
    STATEMENTS1
else
    STATEMENTS_DEFAULT
fi
```

> Check conditions in order. First one that succeeds runs its block. If none succeed, `else` block runs (if present).


```bash
#!/bin/bash

if [ $USER == "root" ]; then
    echo "Welcome, almighty Root. You have full power."
else
    echo "Access limited. Normal mortal detected."
fi
```

- `[ condition ]` → test something.
- `==` → equals.
- `-eq, -lt, -gt` → equal, less than, greater than (for numbers).

Example with numbers:

```bash
x=5
if [ $x -gt 3 ]; then
    echo "$x is greater than 3"
else
    echo "$x is 3 or less"
fi
```

---
## 2️⃣ Else If: Multiple Conditions

```bash
if COMMAND1; then
    STATEMENTS1
elif COMMAND2; then
    STATEMENTS2
elif COMMAND3; then
    STATEMENTS3
else
    STATEMENTS_DEFAULT
fi
```

```bash
#!/bin/bash

read -p "Enter a secret code: " code
if [ "$code" == "123" ]; then
    echo "Door unlocked 🚪"
elif [ "$code" == "007" ]; then
    echo "Welcome, Mr. Bond 🕶️"
else
```


#### Example:

```bash
read -p "Enter a number: " num

if [ $num -gt 0 ]; then
    echo "Positive"
elif [ $num -lt 0 ]; then
    echo "Negative"
else
    echo "Zero"
fi
```

---

## 🛠️ Condition Testing: `[ ]` vs `[[ ]]` vs `test`

There are multiple ways to evaluate conditions:

### 1. Single Brackets `[ ]`

- POSIX-compliant, available everywhere.
- Limited features, requires careful quoting.

Bash

```bash
if [ "$var" = "hello" ]; then ...
```

⚠️ Always quote variables inside `[ ]` to avoid word splitting or empty variable issues.

---

### 2. Double Brackets `[[ ]]` (Bash-specific)

- More powerful, safer, supports pattern matching, regex, logical operators without quoting worries (mostly).
- Not POSIX, so avoid if portability is required.

Bash

```bash
if [[ $var == hello* ]]; then ...
if [[ $num -gt 5 && $str =~ ^[A-Z] ]]; then ...
```

✅ Recommended for Bash-only scripts.

---

### 3. `test` command

Functionally identical to `[ ]`, just written differently.

Bash

```bash
if test -f "/etc/passwd"; then ...
```

Same as:

Bash

```bash
if [ -f "/etc/passwd" ]; then ...
```

---

## 📋 Common Test Operators

### ➤ File Tests

|Operator|Meaning|
|---|---|
|`-e FILE`|File exists|
|`-f FILE`|File is a regular file|
|`-d FILE`|File is a directory|
|`-r FILE`|File is readable|
|`-w FILE`|File is writable|
|`-x FILE`|File is executable|
|`-s FILE`|File has size > 0|

### ➤ String Tests

|Operator|Meaning|
|---|---|
|`-z STRING`|String is empty|
|`-n STRING`|String is not empty|
|`STRING1 = STRING2`|Strings are equal|
|`STRING1 != STRING2`|Strings are not equal|
|`STRING1 < STRING2`|String1 sorts before String2 (lexicographically) — use `[[ ]]`|

> ⚠️ Inside `[ ]`, `<` and `>` must be escaped or quoted to avoid redirection: `[ "$a" \< "$b" ]`

### ➤ Numeric Tests

|Operator|Meaning|
|---|---|
|`INT1 -eq INT2`|Equal|
|`INT1 -ne INT2`|Not equal|
|`INT1 -lt INT2`|Less than|
|`INT1 -le INT2`|Less or equal|
|`INT1 -gt INT2`|Greater than|
|`INT1 -ge INT2`|Greater or equal|

> Use these for integers. For floating point, use `bc` or `awk`.

---

## 💡 Advanced Examples

### ➤ Nested `if`

Bash

```bash
if [ -f "$file" ]; then
    if [ -r "$file" ]; then
        echo "File exists and is readable."
    else
        echo "File exists but is NOT readable."
    fi
else
    echo "File does not exist."
fi
```

---

### ➤ Using `elif` for Multiple Cases

Bash

```bash
case $grade in
    A) echo "Excellent!" ;;
    B) echo "Good!" ;;
    C) echo "Fair." ;;
    *) echo "Needs improvement." ;;
esac
```

> Sometimes `case` is cleaner than long `elif` chains for simple value matching.

But for complex conditions, stick with `if/elif`.

---

### ➤ Logical Operators: `&&` and `||`

You can combine conditions using `&&` (AND) and `||` (OR) — especially inside `[[ ]]`.

Bash

```bash
if [[ $age -ge 18 && $age -le 65 ]]; then
    echo "Working age."
fi

if [[ $user == "admin" || $user == "root" ]]; then
    echo "Privileged user."
fi
```

Inside `[ ]`, you need `-a` (and) and `-o` (or), but they’re deprecated and error-prone — prefer `[[ ]]`.

---

## 🎯 Best Practices & Tips

### ✅ Quote Variables

Always quote variables unless you have a specific reason not to.

Bash

```bash
# ❌ Risky
if [ $name = "John" ]; then ...

# ✅ Safe
if [ "$name" = "John" ]; then ...
```

If `$name` is empty or contains spaces, unquoted version breaks.

---

### ✅ Prefer `[[ ]]` over `[ ]` in Bash

More features, fewer gotchas.

Bash

```bash
# Better
if [[ $file == *.txt ]]; then ...

# Instead of fragile [ $file = *.txt ] which expands glob
```

---

### ✅ Use `(( ))` for Arithmetic

For numeric comparisons, `(( ))` is often more natural:

Bash

```bash
if (( age >= 18 && age <= 65 )); then
    echo "Adult working age."
fi
```

It’s designed for math — no need for `-gt`, `-lt`, etc.

---

### ✅ Keep Conditions Simple

Avoid overly complex `if` statements. Break them into functions or use helper variables.

Bash

```bash
is_valid_user() {
    [[ $1 =~ ^[a-z_][a-z0-9_]{0,31}$ ]]
}

if is_valid_user "$username"; then ...
```

---

## 🐞 Common Pitfalls

### 1. Missing Spaces in `[ ]`

Bash

```bash
# ❌ WRONG
if [$var = "hello"]; then ...

# ✅ CORRECT
if [ "$var" = "hello" ]; then ...
```

`[` is a command — it needs whitespace after it!

---

### 2. Confusing Assignment and Comparison

Bash

```bash
# ❌ This assigns, doesn't compare!
if [ $a = $b ]  # OK for comparison
if [ $a = $b ]  # Still OK
if [[ $a == $b ]] # Also OK

# But this is assignment (inside (( )):
if (( a = b )); then ...  # Assigns b to a, then tests if result ≠ 0
```

Use `==` or `-eq` for comparison.

---

### 3. Using `=` vs `==`

In `[ ]`, `=` is standard. `==` is a bash extension.

In `[[ ]]`, both work, but `==` is more common.

---

## 📊 Summary Table

|Structure|Purpose|Notes|
|---|---|---|
|`if`|Run block if condition true|Always ends with `fi`|
|`else`|Run if previous condition(s) false|Optional|
|`elif`|“Else if” — chain multiple conditions|Can have many; evaluated top-down|
|`[ ]`|POSIX test command|Quote variables, escape `<`, `>`|
|`[[ ]]`|Bash extended test|Safer, supports patterns, regex, no quoting needed usually|
|`(( ))`|Arithmetic evaluation|Great for numeric logic|

---

## 🧪 Practical Script Example

Bash

```bash
#!/bin/bash

read -p "Enter username: " user
read -p "Enter age: " age

# Validate input
if [[ -z "$user" ]]; then
    echo "Error: Username cannot be empty." >&2
    exit 1
fi

if ! [[ "$age" =~ ^[0-9]+$ ]]; then
    echo "Error: Age must be a number." >&2
    exit 1
fi

# Logic with elif
if (( age < 13 )); then
    group="Child"
elif (( age < 20 )); then
    group="Teen"
elif (( age < 65 )); then
    group="Adult"
else
    group="Senior"
fi

echo "User '$user' is in the '$group' group."
```

---
## 3️⃣ Loops: Repetition

Loops let your script _keep doing stuff until told to stop_.

### For Loop

Repeats a fixed number of times:

```bash
for i in {1..5}
do
    echo "Hack attempt #$i"
done
```

Output:

text

```
Hack attempt #1  
Hack attempt #2  
...  
Hack attempt #5
```

---
### While Loop

Repeats **while** the condition is true:

Bash

```bash
count=1
while [ $count -le 3 ]
do
    echo "Looping... ($count)"
    ((count++))
done
```

Output:

text

```
Looping... (1)
Looping... (2)
Looping... (3)
```

---
### Until Loop

Opposite of `while`: keeps looping **until** condition becomes true.

```bash
number=1
until [ $number -gt 3 ]
do
    echo "Number = $number"
    ((number++))
done
```

---
## 4️⃣ Case Statement: Switchboard of Options

Instead of a mess of `if... elif... elif...` you can use `case`.

```bash
#!/bin/bash

read -p "Choose [start/stop/restart]: " action

case $action in
    start)
        echo "System starting..."
        ;;
    stop)
        echo "System stopping..."
        ;;
restart)
        echo "System rebooting..."
        ;;
    *)
        echo "Unknown command!"
        ;;
esac
```

---

### ⚡ Summary 

- **if / else** → decisions.
- **elif** → extra conditions.
- **for** → repeat over items or numbers.
- **while / until** → keep looping by condition.
- **case** → cleaner menu-style branching.

---

