# 🔥 Anatomy of `case`

General form:

```bash
case WORD in
  PATTERN1)
    COMMANDS
    ;;
  PATTERN2 | ALT_PATTERN)
    COMMANDS
    ;;
  PATTERNN)
    COMMANDS
    ;;
  *)
    DEFAULT_COMMANDS
    ;;
esac
```

- **WORD** → what you’re testing (variable, input, filename, `$(command substitution)`).
- **PATTERN** → wildcard/glob patterns (`*`, `?`, `[chars]`, etc.), not regex.

Note: Bash `case` patterns behave like **filename globbing**, NOT like `grep -E` regex! That’s a subtle but crucial hacker fact.

---
# 🔑 Pattern-Matching Powers in `case`

### 1. Wildcard `*`

- Matches **anything** (zero or more chars).
- Often used for the “default” branch.

```bash
case $var in
    start*) echo "It begins with 'start'" ;;
    end*)   echo "It begins with 'end'" ;;
    *)      echo "Default case" ;;
esac
```

---
### 2. Question mark `?`

- Matches **exactly one** character.

```bash
case $code in
    A?)   echo "Two-character code starting with A" ;;
    ??)   echo "Exactly two characters long" ;;
esac
```

---
### 3. Character classes `[...]`

- Works just like globbing in filenames.

```bash
case $digit in
    [0-3]) echo "Low digit" ;;
    [4-6]) echo "Middle digit" ;;
    [7-9]) echo "High digit" ;;
    *)     echo "Not a valid digit" ;;
esac
```

---
### 4. Alternation `|`

- Let’s you combine multiple patterns into one branch, like “OR.”

```bash
case $animal in
    cat|kitty) echo "It’s a feline." ;;
    dog|puppy) echo "It’s a canine." ;;
    *) echo "Some other beast." ;;
esac
```

---
### 5. Escaping special characters

If your pattern needs to be literal (like matching `*` or `?` itself), you escape with `\` or `[]`.

```bash
case $symbol in
    \*) echo "It's literally an asterisk" ;;
    [?]) echo "It’s a question mark" ;;
esac
```

---
# 💡 Advanced Techniques

### ✅ Nested case

You _can_ nest a `case` inside another, though it’s usually cleaner to flatten logic:

```bash
case $env in
    prod)
        case $service in
            web) echo "Deploying web to prod" ;;
            db)  echo "Deploying DB to prod" ;;
        esac
        ;;
    dev)
        echo "Deploying to dev environment"
        ;;
esac
```

---

### ✅ Case for command‑line options (lightweight parser)

A typical hacker move: parsing arguments.

```bash
while [[ $# -gt 0 ]]
do
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
            echo "Unknown option: $1"
            shift
            ;;
    esac
done
```

Here you loop through all arguments, and `case` branches decide how to handle them.

---
### ✅ Case with command substitution

You don’t _only_ need variables—you can put a command right in:

```bash
case $(date +%u) in
    6|7) echo "It’s the weekend—hacking at leisure!" ;;
    *)   echo "It’s a weekday—hack efficiently!" ;;
esac
```

(`%u` gives day of week as 1=Monday … 7=Sunday.)

---
# ⚔️ Professional Hacker’s Tips

- **Order matters**: the first matching pattern wins. Bash won’t check further branches once one matches.
- **Default safety**: always have a `* )` case for unexpected input (never trust user input!).
- **Combine OR patterns with `|`** instead of duplicating logic.
- **Case vs if-elif**: Use `case` if you expect a neat set of options. Use `if` when conditions involve arithmetic or complex expressions.

---
# 🚀 Hacker‑style real examples

**1. File categorizer script**

```bash
for f in *; do
    case $f in
        *.sh)   echo "$f → Shell script" ;;
        *.txt)  echo "$f → Text file" ;;
        *.jpg|*.png) echo "$f → Image file" ;;
        *)      echo "$f → Unknown type" ;;
    esac
done
```

**2. Services switchboard**

```bash
read -p "Choose service (web/db/cache): " svc
case $svc in
    web)   systemctl restart nginx ;;
    db)    systemctl restart postgresql ;;
    cache) systemctl restart redis ;;
    *)     echo "Unknown service!" ;;
esac
```

---
# 🎯 Challenge for you

Write a `case` script that takes a user’s input for a **file extension** (`txt`, `log`, `sh`, `jpg`) and:

- Prints a playful description of the file type.
- If input is `exit`, the script should gracefully end.
- Anything else? Give them a cheeky “Unknown file type” message.

---

