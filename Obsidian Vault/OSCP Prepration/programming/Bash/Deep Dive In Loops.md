## 🔁 The Two Classic Loop Types in Bash

Think of loops as two flavors of hacker gadgetry:

### 1. **`for` loops**

This is like a “mission control checklist.” You tell the loop exactly what items to iterate over, and it runs commands for each.

**Basic syntax:**

```bash
for variable in list
do
    command(s)
done
```

**Example:**

```bash
for file in *.txt
do
    echo "Processing $file"
done
```

What this does: looks at every `.txt` file in the current directory and prints its name. Imagine batch‑renaming or scanning all your target files.

👉 Variations:

- Counting numbers:
    
    ```bash
    for i in {1..5}
    do
        echo "Iteration $i"
    done
    ```
    
- Using the `seq` command for flexible ranges:
    
    ```bash
    for i in $(seq 1 2 10)
    do
        echo "Number $i"
    done
    ```
    
    (This loops from 1 to 10, stepping by 2.)

---
### 2. **`while` loops**

This is like a guard at the dungeon gate—you keep looping _while_ some condition is true. When it fails, the loop breaks.

**Basic syntax:**

```bash
while condition
do
    command(s)
done
```

**Example:**

```bash
count=1
while [ $count -le 5 ]
do
    echo "Loop number $count"
    ((count++))
done
```

Here, as long as `$count` is ≤ 5, the loop continues. Once `$count` exceeds 5, the guard goes, “Nope, you’re done.”

---
### 3. **The endless loop (hacker’s “heartbeat”)**

Not for the faint of heart—will run _forever_ unless you break manually (`Ctrl+C`) or include logic inside.

```bash
while true
do
    echo "Still alive..."
    sleep 1
done
```

This is how you can mock up “persistent monitoring” or keep a script running in the background.

---

## 📚 Useful control commands _inside_ loops

- `break` ⇒ smash out of the loop early.
- `continue` ⇒ skip the rest of this cycle, jump back to the top.

**Example:**

```bash
for i in {1..10}
do
    if [ $i -eq 5 ]; then
        echo "Skipping 5"
        continue
    fi
    if [ $i -eq 8 ]; then
        echo "Stopping at 8"
        break
    fi
    echo "Number $i"
done
```

---
## 🚀 Real hacker‑ish examples

1. **Quick port scanner (toy example):**

```bash
for port in {20..25}
do
    nc -zv 127.0.0.1 $port
done
```

(Tries ports 20–25 on localhost. Replace target IP to scan elsewhere.)

2. **Batch renamer:**

```bash
count=1
for f in *.jpg
do
    mv "$f" "image_${count}.jpg"
    ((count++))
done
```

3. **Monitoring a log file until a keyword appears:**

```bash
while true
do
    grep "ERROR" server.log && echo "Error found!" && break
    sleep 2
done
```

---
### Hacker’s mindset tip

Think of loops as **automation spells**. `for` loops are laser‑focused (iterate over a known set), `while` loops are conditional traps (run until situation changes). When combined with file operations, text processing, or tools like `awk`/`grep`, you wield scary efficiency.

---




