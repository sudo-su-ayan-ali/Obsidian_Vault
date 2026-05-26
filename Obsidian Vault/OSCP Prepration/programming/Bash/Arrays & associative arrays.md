# 🔹 Part 1 — Indexed Arrays (numeric keys)

### Declaring

```bash
# Method 1: declare 
declare -a fruits  
# Method 2: assign directly 
fruits=("apple" "banana" "cherry")
```

### Accessing

```bash
echo "${fruits[0]}"     # apple 
echo "${fruits[2]}"     # cherry
```

### All elements

```bash
echo "${fruits[@]}"     # apple banana cherry 
echo "${fruits[*]}"     # apple banana cherry
```

⚠️ Difference:

- `"$@"` → expands each element separately (preserves spaces).
    
- `"$*"` → expands as a single string.
    

### Indexes

```bash
echo "${!fruits[@]}"    # 0 1 2
```

### Length

```bash
echo "${#fruits[@]}"    # 3
```

### Adding & Modifying

```bash
fruits[3]="date"        # add new element 
fruits[1]="blueberry"   # modify existing
```

### Iterating

```bash 
for fruit in "${fruits[@]}"; do
	echo "$fruit"
done
```

---

# 🔹 Part 2 — Associative Arrays (string keys)

Introduced in **Bash 4+**.

### Declaring

```bash
declare -A capitals 
capitals=(   
	[France]="Paris"   
	[Germany]="Berlin"   
	[Italy]="Rome" 
)
```

### Accessing

```bash
echo "${capitals[France]}"    # Paris
```

### Keys & Values

```bash
echo "${!capitals[@]}"        # France Germany Italy 
echo "${capitals[@]}"         # Paris Berlin Rome
```

### Length

```bash
echo "${#capitals[@]}"        # 3
```

### Adding & Modifying

```bash
capitals[Spain]="Madrid" 
capitals[Germany]="Hamburg"   # modifies existing
```

### Iterating

```bash
for country in "${!capitals[@]}"; do
	echo "$country -> ${capitals[$country]}"
done
```

---

# 🔹 Part 3 — Advanced Array Tricks

### Slice (indexed only)

```bash
fruits=("apple" "banana" "cherry" "date" "elderberry") 
echo "${fruits[@]:1:3}"     # banana cherry date
```

### Replace element

```bash
fruits=("a b" "c d" "e f") echo "${fruits[0]/b/B}"     # a B
```

### Concatenation

```
all=("${fruits[@]}" "${capitals[@]}")
```

### Sorting (with `mapfile` and `sort`)

```bash
mapfile -t sorted < <(printf "%s\n" "${fruits[@]}" | sort) printf '%s\n' "${sorted[@]}"
```

---

# 🔹 Part 4 — Practical Use Cases

1. **Counting word frequencies (assoc array)**:
    

```bash
declare -A count
for word in $(cat file.txt); do
	((count[$word]++))
done
for w in "${!count[@]}"; do
	echo "$w: ${count[$w]}"
done
```

2. **Mapping services to ports**:
    

```bash
declare -A services=(
	[http]=80
	[https]=443
	[ssh]=22
)
echo "SSH runs on port ${services[ssh]}"
```

3. **Batch processing files**:
    

```bash
files=("file1.log" "file2.log") 
for f in "${files[@]}"; do   
	gzip "$f" 
done
```

---
# 🔹 Quick Summary

- **Indexed arrays** → numeric keys, use `declare -a`.
    
- **Associative arrays** → string keys, use `declare -A` (Bash ≥ 4).
    
- Use `"${!array[@]}"` for keys, `"${array[@]}"` for values.
    
- Arrays support slicing, iteration, and substitution tricks.
    
- Associative arrays are perfect for lookups, mappings, counters.

---

