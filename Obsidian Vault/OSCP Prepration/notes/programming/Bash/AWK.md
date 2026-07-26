## **1. What is `awk`?**

- `awk` = **pattern scanning and processing language**.
    
- Reads input line by line, splits it into **fields** (columns), then applies **rules** in the form:
    

`pattern { action }`

- If `pattern` matches, the `{action}` executes.
    
- Fields are `$1`, `$2`, … `$NF` (number of fields).
    
- `$0` = the entire line.
    

---

## **2. Basic Usage**

- Default action = `print $0` (print whole line).
    
```bash
awk '{print}' file              # print all lines 
awk '{print $1}' file           # print first column 
awk '{print $1, $3}' file       # print first and third columns 
awk -F: '{print $1}' /etc/passwd   # set delimiter to “:”
```

---

## **3. Patterns**

- Run action only if condition matches:
    

```bash
awk '$3 > 1000 {print $1}' /etc/passwd   # users with UID > 1000 
awk '$1 == "ERROR" {print $0}' logfile   # print ERROR lines
```

- Regex match with `~`:
    

```bash
awk '$1 ~ /^192\.168\./ {print $0}' ips.txt awk '$1 !~ /root/ {print $0}' /etc/passwd
```


---

## **4. Built-in Variables**

- `NR` → record number (line number across input)
    
- `FNR` → line number relative to file
    
- `NF` → number of fields
    
- `FS` → input field separator (default = space)
    
- `OFS` → output field separator (default = space)
    
- `RS` → record separator (default = newline)
    
- `ORS` → output record separator (default = newline)
    

Examples:

```bash
awk '{print NR, $0}' file      # print line numbers 
awk '{print $NF}' file         # last column 
awk 'BEGIN {FS=","} {print $2}' file.csv 
awk 'BEGIN {OFS=" - "} {print $1,$2}' file.txt
```

---

## **5. BEGIN and END Blocks**

- `BEGIN` runs once before input.
    
- `END` runs once after input.
    

```bash
awk 'BEGIN {print "Start"} {print $1} END {print "Done"}' file
```

Useful for headers, initialization, summaries.

---

## **6. Arithmetic and Strings**

- Arithmetic operators: `+ - * / % ^`
    
- String concatenation: just put strings side by side.
    

Examples:

```bash
awk '{sum += $2} END {print "Total:", sum}' sales.txt 
awk '{print $1, length($1)}' words.txt 
awk '{print toupper($1), tolower($2)}' file
```

---

## **7. Control Structures**

Awk has if/else, loops, and functions like a mini programming language.

### If

```bash
awk '{if ($3>50) print $1,"HIGH"; else print $1,"LOW"}' data.txt
```

### Loops

```bash
awk '{for (i=1;i<=NF;i++) print $i}' file
```

### User-defined functions

```bash
awk 'function square(x) {return x*x} {print $1, square($1)}' nums.txt
```

---

## **8. Arrays**

- Associative arrays (string-indexed).
    

```bash
awk '{count[$1]++} END {for (w in count) print w, count[w]}' words.txt
```

Example: word frequency counter.

---

## **9. Input Splitting & Delimiters**

- Default separator = whitespace.
    
- Use `-F` for simple separators, or set `FS` inside script.
    

```bash
awk -F, '{print $2}' file.csv awk 'BEGIN {FS="[,:]"} {print $1,$2}' file   # split on comma OR colon
```

---

## **10. Practical Examples**

### Count lines

```bash
awk 'END {print NR}' file
```

### Sum numbers in column 2

```bash
awk '{sum+=$2} END {print sum}' data.txt
```

### Average of column 3

```bash
awk '{sum+=$3; n++} END {print sum/n}' data.txt
```

### Extract IPs from Apache logs

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

### Reformat date

```bash
echo "2025-09-14" | awk -F- '{print $3 "/" $2 "/" $1}' # 14/09/2025
```

---

## **11. Advanced Features**

### Multi-file handling

```bash
awk '{print FILENAME, FNR, $0}' file1 file2
```

### In-place editing (GNU awk only)

```
gawk -i inplace '{sub(/foo/,"bar")}1' file
```

### External variables

```
awk -v limit=100 '$3 > limit {print $1}' data.txt
```

### Using system commands

```bash
awk '{print $1 | "sort"}' file
```

---

## **12. Real-World Scripts**

### 1. Disk usage report

```bash
df -h | awk 'NR>1 {print $1, $5}'
```

### 2. CSV column statistics

```bash
awk -F, '{sum+=$3; if($3>max) max=$3} END {print "Sum:",sum,"Max:",max}' data.csv
```

### 3. Log filtering

```bash
awk '$9 ~ /^[45]/ {print $1,$9}' access.log
```

(Print IPs with 4xx/5xx HTTP codes.)

### 4. Group by field

```bash
awk '{bytes[$1]+=$2} END {for(ip in bytes) print ip, bytes[ip]}' traffic.log
```

---
