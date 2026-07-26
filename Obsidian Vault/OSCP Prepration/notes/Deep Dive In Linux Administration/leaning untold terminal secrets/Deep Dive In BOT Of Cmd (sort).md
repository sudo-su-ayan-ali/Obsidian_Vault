### 🧾 What `sort` does

The `sort` command takes _text input_ (from a file or from another command) and arranges the lines in a specific order.

**By default:**

- It sorts alphabetically (lexicographically).
- It’s case-sensitive (so `A` comes before `a`).

---

### 🔨 Basic usage

- **`sort filename`**  
    Reads the contents of _filename_ and spits it out sorted.  
    Example:
    
    ```bash
    cat guests.txt
    Zoe
    alex
    Mike
    bob
    ```

    ```bash
    sort guests.txt
    Mike
    Zoe
    alex
    bob
    ```
    
    (Notice uppercase letters come before lowercase by default!)

---
### 💎 Common options (treasure chest of variations)

- **`-r`** → reverse order
    
    text
    
    ```
    sort -r filename
    ```
    
    Puts the list upside down.
    
- **`-f`** → ignore case while sorting  
    So `Zoo` and `apple` will be compared equally without case bias.
    
- **`-n`** → numeric sort  
    Without this, `10` would come _before_ `2` (since it goes character by character).
    
	text
	
	```
	sort -n numbers.txt
	```
	
	Now, glorious numerical order is restored.
- **`-k`** → sort by a specific column (a key)  
    Suppose you’ve got:
    
    ```bash
    Alice  23
    Bob    19
    Carol  41
    ```
    
    Command:
    
    ```bash
    sort -k2 -n data.txt
    ```
    
    → Sorts by age, numerically.
    
- **`-u`** → unique (removes duplicates)  
    Think of it as decluttering a hacker’s messy desk.
    

---
### ⚔️ Chaining with pipes

Where `sort` becomes _epic hacker material_ is when piping other commands into it.  
Example: sorting all system users alphabetically:

text

```
cat /etc/passwd | cut -d: -f1 | sort
```

Boom. Spyglass view of users, perfectly arranged.

Or, ranking running processes by memory usage and then sorting:

text

```
ps aux | sort -k4 -n
```

Now it lists processes by memory consumption (ascending). Add `-r` to flip into descending order (BIGGEST hogs on top).

---
💡 **Pro Hacker Tip:** Combine `sort` with `uniq` for maximum cleanup. For instance:

text

```
cat logfile.txt | sort | uniq -c
```

This counts repeated lines and tells you how many times each occurs. Great for spotting repeated intrusions, most common IPs, etc.

---

So, fellow apprentice, the challenge is this:  
Create a little text file with random words or numbers in it, then practice sorting them with different flags (`-n`, `-r`, `-f`, etc). Observe how the output shifts.

It’s like learning how to flip a digital deck of cards until the order clicks.

---
