
# ⭐ Why You Don’t Need a “Perfect” Dictionary

There is **no one perfect wordlist**.  
Different websites use different naming patterns.  
So the trick is knowing **which list to use for which situation**.

---

# ⭐ The 3 Best Wordlists for OSCP / Realistic CTFs

### **1. `directory-list-2.3-medium.txt`** (MOST USED)

Path on Kali:

```
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Use this **90% of the time**.  
It’s perfect for OSCP-style machines.

---

### **2. `common.txt` (SecLists)**

Path:

```
/usr/share/seclists/Discovery/Web-Content/common.txt
```

Use this when:

- You want a **fast scan**
    
- You want **quick low-hanging directories**
    

---

### **3. `raft-medium-words.txt` (SecLists)**

Path:

```
/usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
```

Use when the website structure looks _realistic_ (enterprise apps).

---

# ⭐ How to Choose the Right One (Super Simple Rule)

### ✔ **Small site → use common.txt**

### ✔ **Normal OSCP/CTF site → use directory-list-2.3-medium.txt**

### ✔ **Enterprise-like / multiple directories → use raft-medium-words.txt**

---

# ⭐ Bonus: Use File Extensions (Important!)

Directories often hide files like:

- `.php`
    
- `.html`
    
- `.txt`
    
- `.asp`
    

Add extensions like this:

```
gobuster dir -u http://TARGET -w <wordlist> -x php,txt,html
```

This finds things you would _never_ see otherwise.

---

# ⭐ Still Not Finding Anything? Try Recursive Search

Some sites hide directories deep inside.

Use:

```
gobuster dir -u http://TARGET -w <wordlist> -t 60 -r
```

Or use **dirsearch**, which has recursion by default.

---

# ⭐ Golden Tip: Use This Workflow

### **Step 1: Quick scan**

```
common.txt
```

### **Step 2: Full scan**

```
directory-list-2.3-medium.txt
```

### **Step 3: Heavy scan (if needed)**

```
raft-medium-words.txt
```

This 3-step method works in **almost every OSCP machine**.

---

# ⭐ If You Want…

I can also give you:

✔ Best wordlist for PHP websites  
✔ Best wordlist for login pages  
✔ Best OSCP-specific wordlist  
✔ My custom shortcut command for gobuster/dirsearch

Just tell me what you want next!