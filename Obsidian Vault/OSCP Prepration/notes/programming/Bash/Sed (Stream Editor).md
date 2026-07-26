# 🔹 Deep Dive into `sed` (Stream Editor)

`sed` = **stream editor**, a tool that reads input line by line, applies editing commands, and outputs the result. Unlike normal text editors, it works non-interactively — perfect for scripts, automation, and batch edits.

---

## **1. How `sed` Works**

- Reads input **line by line** (stream).
    
- Each line is matched against an **address** (line number or regex).
    
- If it matches, the **command(s)** are applied.
    
- Prints the result (unless suppressed with `-n`).
    

General syntax:

```bash
sed [options] 'address command' file
```

Common options:

- `-n` → suppress automatic printing (only print when explicitly told).
    
- `-i` → edit file in place.
    
- `-e` → multiple expressions.
    
- `-E` → use extended regex.
    

---

## **2. Basic Substitution**

Substitute text with `s///`.

```bash
sed 's/foo/bar/' file         # replace first occurrence of foo with bar 
sed 's/foo/bar/g' file        # replace all occurrences in each line 
sed 's/foo/bar/2' file        # replace 2nd occurrence only 
sed 's/foo/bar/gI' file       # replace case-insensitive
```

- Delimiters don’t have to be `/`:
    

```bash
sed 's|/usr/local|/opt|' file
```

- Backreferences (regex groups):
    

```bash
echo "hello world" | sed -E 's/(hello) (world)/\2 \1/' # world hello
```

- Use `&` to refer to matched text:
    

```bash
echo "foo" | sed 's/.*/(&)/' # (foo)
```

---

## **3. Addresses**

By default, `sed` applies to all lines. You can target lines by:

- **Line numbers**:
    

```bash
sed '5s/foo/bar/' file        # only line 5 
sed '1,3d' file               # delete lines 1 to 3
```

- **Patterns**:
    

```bash
sed '/^#/d' file              # delete lines starting with # 
sed '/error/s/fail/FAIL/' log # replace only in lines containing "error"
```

- **Ranges**:
    

```bash
sed '/start/,/end/d' file     # delete between patterns
```

---

## **4. Editing Operations**

- **Delete lines**:
    

```bash
sed '2d' file                 # delete 2nd line 
sed '/^$/d' file              # delete blank lines
```

- **Insert & Append**:
    

```bash
sed '2i\Inserted line' file   # insert before line 2 
sed '2a\Appended line' file   # append after line 2
```

- **Change (replace whole line)**:
    

```bash
sed '3c\This is new text' file
```

- **Multiple commands**:
    

```bash
sed -e 's/foo/bar/' -e '/baz/d' file
```

---

## **5. Printing**

By default, `sed` prints all lines. To control:

```bash
sed -n 'p' file               # print all lines (pointless here) 
sed -n '5p' file              # print only line 5 
sed -n '/ERROR/p' log         # print only lines with ERROR
```

---

## **6. Hold and Pattern Space**

`sed` has two spaces:

- **Pattern space** = current line being processed.
    
- **Hold space** = temporary buffer you can store text in.
    

Example:

```bash
sed -n '1h;2{H;x;p}' file
```

- `h` → copy pattern space to hold space
    
- `H` → append to hold space
    
- `x` → swap pattern & hold space
    

This allows multiline editing.

---

## **7. Multiline Processing**

- `N` → append next line into pattern space.
    

```bash
sed 'N;s/\n/ /' file   # join pairs of lines into one line
```

- Delete empty lines followed by another empty:
    

```bash
sed '/^$/N;/^\n$/d' file
```

---

## **8. Advanced Substitution**

- Replace only if match in certain position:
    

```bash
sed 's/^foo/bar/' file       # only at beginning of line 
sed 's/foo$/bar/' file       # only at end of line
```

- Global replace + count:
    

```bash
sed -n 's/foo/bar/gp' file   # replace and print lines that changed
```

- Number substitutions:
    

```bash
sed 's/foo/bar/3' file       # only 3rd match per line
```

---

## **9. Escaping & Special Characters**

- Use `\1`, `\2` for regex groups.
    
- Use `&` for whole match.
    
- Use different delimiters (`|`, `:`) to avoid escaping.
    

Example:

```bash
sed 's:/usr/local:/opt:' file
```

---

## **10. Real Use Cases**

1. **Remove comments & blank lines from config**:
    

```bash
sed -E '/^\s*#/d;/^\s*$/d' config.cfg
```

2. **Prefix line numbers**:
    

```bash
nl file | sed 's/^ *[0-9]\+\t/[&]/'
```

3. **Bulk rename files from `.txt` to `.md`**:
    

```bash
ls *.txt | sed 's/\(.*\)\.txt/mv "&" "\1.md"/' | bash
```

4. **Capitalize words**:
    

```bash
echo "hello world" | sed 's/\b\(.\)/\u\1/g' # Hello World
```

5. **Swap fields (comma-separated)**:
    

```bash
echo "first,last" | sed -E 's/(.*),(.*)/\2,\1/' # last,first
```

6. **Extract specific block** (between two patterns):
    

```bash
sed -n '/BEGIN/,/END/p' file
```

---

## **11. Common Pitfalls**

- Don’t forget quotes around the script (`'s/foo/bar/'`) → otherwise shell eats regex.
    
- GNU vs BSD `sed` differences:
    
    - `-i` works differently on macOS (`sed -i ''`).
        
    - Use `-E` for extended regex (GNU allows `-r`).
        
- Greedy regex can match more than expected → use `.*?` (but basic `sed` doesn’t support lazy, only GNU with `-E` and careful crafting).
    

---

## **12. Performance & Scripting**

- Use `sed` in pipelines for efficiency:
    

```bash
grep "ERROR" log | sed 's/.*ERROR: //'
```

- Use with `find`:
    

```bash
find . -type f -name "*.html" -exec sed -i 's/old/new/g' {} +
```

- Batch editing configs:
    

```bash
sed -i 's/^port=.*/port=8080/' app.conf
```

---

