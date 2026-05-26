### 🖥️ Bash Scripting Overview

1. **Basics**
    
    - Scripts are just text files with commands.
    - Always start with: `#!/bin/bash`.
    - Make executable with `chmod +x script.sh`.
    
2. **Variables & Inputs**
    
    - Store values: `x=10`.
    - Access with `$x`.
    - Gather user input using `read`.
    
3. **Control Flow**
    
    - `if / else`: decisions.
    - `case`: multiple-choice logic.
    - `for` & `while`: loops for repetition.
    
4. **Functions & Arguments**
    
    - Functions group commands for reuse.
    - Script arguments: `$1, $2, …`, `$@`.
    
5. **Files & Redirection**
	- Control outputs: `>`, `>>`, `2>`.
    - Read files line-by-line.
    - Manage logs.
    
6. **Process / Permissions / Scheduling**
    
    - Exit codes with `$?`.
    - Permission handling (`chmod`, `chown`).
    - Automate tasks with `cron` (scheduled scripts).
    
7. **Advanced Tools**
    
    - Arrays & associative arrays.
    - Text manipulation with `grep`, `sed`, `awk`.
    - Command substitution `$(...)`.