## 1️⃣ Variables: Storing and Recalling Data

A **variable** is like a labeled box where you can store stuff (numbers, text, etc.), then reuse it.

### Create a variable

```bash
name="Neo"
```

⚠️ No spaces around `=` — Bash is allergic to them. `name = "Neo"` will break.

### Access a variable

```bash
echo $name
```

Output:

text

```
Neo
```

---
## 2️⃣ Types of Variables

Bash doesn’t care if it’s text or numbers — _everything_ is basically text unless you force it to act like math.

- **String example:**
    
    ```bash
    hacker="Trinity"
    echo "Hello $hacker"
    ```
    
- **Arithmetic example:**
    
    ```bash
    x=5
    y=3
    echo $((x + y))
    ```
    
    Output:
    
    text
    
    ```
    8
    ```
    

---
## 3️⃣ Input: Asking the User for Data

This is where it feels like the script is _interviewing_ you.

- Using `read`:
    
    ```bash
    #!/bin/bash
    read -p "Enter your codename: " codename
    echo "Welcome, Agent $codename."
    ```
    

Run it:

text

```
Enter your codename: Ghost
Welcome, Agent Ghost.
```

### 🔍 What’s happening?

- `#!/bin/bash` — Shebang line: tells the system to run this script with Bash.
- `read -p "Enter your codename: " codename`
    - `read` — built-in command to capture user input.
    - `-p` — flag that lets you display a prompt _on the same line_ as the input cursor.
    - `"Enter your codename: "` — the prompt message shown to the user.
    - `codename` — variable name where the user’s input will be stored.
- `echo "Welcome, Agent $codename."` — prints a personalized welcome message using the stored input.
---

### 🚀 Pro Tips:

1. **Timeout Input (optional):**
    
    ```bash
    read -t 10 -p "Enter your codename (10 sec timeout): " codename
    ```
    
    User has 10 seconds to respond — great for automated systems.
    
2. **Silent Input (for passwords or secrets):**
    
    ```bash
    read -s -p "Enter your access code: " access_code
    echo  # add newline after silent input
    ```

1. **Default Value if Empty:**
    
    ```bash
    read -p "Enter your codename [default: Spectre]: " codename
    codename=${codename:-Spectre}
    ```
    
1. **Validate Input? Try a loop:**
    
    ```bash
    while [[ -z "$codename" ]]; do
        read -p "Codename cannot be empty. Enter again: " codename
    done
    ```
    

---


## 4️⃣ Command-Line Arguments

Sometimes you don’t _ask_ users for input — they pass it when running the script.

Example `agent.sh`:

```bash
#!/bin/bash
echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
```

Running:

```bash
./agent.sh Matrix Neo
```

Output:

```bash
Script name: ./agent.sh
First argument: Matrix
Second argument: Neo
```

Cheat sheet:

- `$0` → script name
- `$1, $2, ...` → arguments
- `$@` → all arguments

---
### ⚡ Summary 

- `variable="value"` → store data (no spaces around `=`).
- `$variable` → access value.
- `$(())` → do arithmetic.
- `read` → ask for input.
- `$1, $2...` → command-line arguments.

---

