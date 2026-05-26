## 1️⃣ What is a Bash Script?

- A **bash script** is basically a text file containing commands you’d normally type in the terminal.
- Instead of running commands one by one, you put them in a file and tell Bash to execute them in order.
- Handy for automation, shortcuts, and… looking cool in movies when code scrolls on the screen 🕶️.

---

## 2️⃣ The Shebang (`#!/bin/bash`)

- Every script usually starts with:
    
    ```bash
    #!/bin/bash
    ```
    
- This first line is called the **shebang**. It tells the system:  
    “Use **bash** to run the commands below.”
- If you skipped it, Linux might try using a _different_ shell. That’s like asking a dog to play piano — it’ll bark at you instead of performing.

---
## 3️⃣ Writing Your First Script

Make a file named `hello.sh`:

```bash
#!/bin/bash
echo "Hello, hacker!"
```

---
## 4️⃣ Making It Executable

By default, new files don’t have permission to “run.” You must give them execution rights with `chmod`:

```bash
chmod +x hello.sh
```

Now you can run it:

```bash
./hello.sh
```

Output:

text

```
Hello, hacker!
```

💡Notice the `./` in front? It means “run this script from the current folder.”

---
## 5️⃣ Comments

Use `#` to leave notes in your script. They don’t get executed:

```bash
#!/bin/bash
# This is a comment
echo "This line will run"
# echo "This line is ignored"
```

---

### ⚡ Summary 

- **Shebang** (`#!/bin/bash`) → tells the system to use bash.
- **echo** → prints text.
- **chmod +x** → make script executable.
- **./script.sh** → run the script.
- **#** → comment your code.

---


