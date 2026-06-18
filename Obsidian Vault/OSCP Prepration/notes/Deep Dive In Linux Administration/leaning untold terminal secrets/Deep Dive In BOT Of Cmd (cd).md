## 🌲 The Map: File System Structure

- **Root (`/`)** → the very top of the filesystem tree (everything hangs from here).
- **Home (`~`)** → your "personal lair," typically `/home/username`. Almost every journey begins here.

---

## 🔑 Using `cd`

Technically, the formula is:

text

```
cd [path]
```

- **`path`** can be **absolute** (from the very top `/`) or **relative** (from where you _currently_ are).

---
### ⚙️ Major Moves

1. **`cd folderName`**  
    Goes _inside_ that folder (assuming it exists in your current spot).
    
    - Example:
        
        text
        
        ```
        cd Documents
        ```
        
2. **`cd ..`** (_dot dot magic_)  
    Move _up one level_ (to the parent directory). This is like climbing up one branch of the tree.
    
3. **`cd .`**  
    This means “stay exactly where you are.” Doesn’t really move you—mostly a philosophical reminder.
    
4. **`cd ~`**  
    Teleports you back **home** instantly (like typing `/home/yourUserName`).
    
5. **`cd -`**  
    **Toggle back** to the _previous_ directory. Super useful when bouncing between two locations.
    
6. **`cd /absolute/path`**
    
    - Starts from the root (`/`).
    - Example: `cd /etc/ssh` takes you _directly_ into that directory, no matter where you started.

---
### 🧭 Practical Examples

Imagine this simple structure:

text

```
/home/user/
│
├── Documents/
│   ├── project.txt
│
└── Downloads/
    ├── song.mp3
```

- Start in `/home/user/`.
- `cd Documents` → now in `/home/user/Documents/`.
- `cd ..` → back at `/home/user/`.
- `cd Downloads` → now in `/home/user/Downloads/`.
- `cd ..` → back home.
- `cd -` → jumps back into Downloads (since that was your “previous” spot).

---
### 🧙 Hacker Sensei Extra Tips

- Combine with `pwd` (print working directory) to always know _where_ your boots left you.
- Remember tab-completion: type `cd Do` + press `TAB` → it auto-completes to `Documents/` if unambiguous. Lazy but powerful.
- If the target has spaces in the name, quote it:
    
    text
    
    ```
    cd "My Folder"
    ```
    
- If you ever feel lost, just type:
    
    text
    
    ```
    cd ~
    ```
    
    Boom. Safe at home.

---

#### ***NOTE*** 
`ls` = see the battlefield; `cd` = move across it.
