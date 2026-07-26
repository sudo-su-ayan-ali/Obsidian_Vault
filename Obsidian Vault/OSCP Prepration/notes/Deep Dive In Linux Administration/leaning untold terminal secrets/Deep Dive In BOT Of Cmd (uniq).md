### 🔍 What `uniq` Does

- It looks at **adjacent lines** in a file or from input, and **removes duplicates**.
- Important word: **adjacent**. If duplicates aren’t next to each other, `uniq` won’t notice. That’s why it’s often paired with `sort` (which lines things up neatly before filtering).

---
### ⚔️ Basic Usage


```bash
uniq file.txt
```

- Reads `file.txt` and prints it with consecutive duplicates squashed into a single copy.

Example `file.txt`:


```bash
alpha
alpha
beta
beta
beta
gamma
alpha
```

See? The first “alpha alpha” got merged, the “beta” triplets compressed into one, but that sneaky last “alpha” stayed because it wasn’t **next to** the others.

---
### 🛠 Frequently Used Options

- **`-c`** → counts occurrences.
    
    ```bash
    uniq -c file.txt
    ```
    
    Output might look like:
    
    
    ```bash
          2 alpha
          3 beta
          1 gamma
          1 alpha
    ```
    
    Hackers love this for quick pattern frequency analysis.
    
- **`-d`** - → print _only_ duplicated lines (ignores unique ones).
    
- **`-u`** → print _only_ unique (non-duplicated) lines.
    
- **`-i`** → ignore case differences (treat `Alpha` and `alpha` as the same).
    

---
### 🔗 Classic Hacker Combo: Sorting + Uniquing

Since `uniq` only catches _adjacent_ duplicates, we usually sort first:


```bash
sort file.txt | uniq -c
```

This combo gives you a nice **frequency table** of all entries in the file. Think of it as doing reconnaissance: spotting the most common IPs, usernames, or errors in a log file.

---
### 🕶 Hacker Flavor Example

Imagine you’ve got a log of failed SSH attempts:


```bash
192.168.0.15
192.168.0.15
45.67.123.9
192.168.0.15
45.67.123.9
```

Run:


```bash
sort sshfails.log | uniq -c
```

Result:

```bash
      3 192.168.0.15
      2 45.67.123.9
```

Now you instantly know which IP’s trying hardest to brute-force your lair. Easy intel.

---
So—`uniq` = the **noise-breaker**. Used raw, it’s handy, but paired with `sort`, it becomes a laser scalpel for logs and data.