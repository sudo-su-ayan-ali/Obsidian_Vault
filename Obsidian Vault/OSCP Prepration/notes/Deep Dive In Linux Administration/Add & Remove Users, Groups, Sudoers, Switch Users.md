# 🏗️ Users and Groups in Linux

Linux organizes people into two structures:

- **Users** = individual accounts (like Alice, Bob).
- **Groups** = collectives of users (like developers, designers). Enables team-based permissions.
- **root** = the supreme wizard; has access to _everything_ (careful with this one).

---

## ➕ Adding and Removing Users

- **Add a user:**
    
    text
    
    ```
    sudo adduser alice
    ```
    
    Creates a new user _alice_, home directory `/home/alice`, and assigns her a private group named `alice`. (Interactive—it’ll ask for password etc.)
    
    Alternatively, some distros use:
    
	text
    
    ```
    sudo useradd -m alice
    ```
    
    (`-m` ensures home directory is created).
    
- **Delete a user:**
    
    text
    
    ```
    sudo deluser alice
    ```
    
    or (depending on distro):
    
	 text
    
    ```
    sudo userdel -r alice
    ```
    
    `-r` removes their home directory too. (Nukes from orbit 🚀).
    

---
## 👥 Adding and Removing Groups

- **Add a group:**
    
    text
    
    ```
    sudo groupadd hackteam
    ```
    
- **Delete a group:**
    
    text
    
    ```
    sudo groupdel hackteam
    ```
    
- **Add a user to a group:**
    
    text
    
    ```
    sudo usermod -aG hackteam alice
    ```
    
    This means "append (`-a`) Alice into supplementary Group `hackteam`."  
    Without `-a`, you overwrite all their groups (rookie mistake = Alice gets banished from her old teams).
    
- **Check user’s groups:**
    
    text
    
    ```
    groups alice
    ```
    
    or if logged in as Alice:
    
     text
    
    ```
    groups
    ```
    

---

## 🧙 Sudoers (Root Superpowers)

By default, **only root** can run system-level commands like adding/removing users. Normal users live in safety lanes. But we can give a chosen few _temporary god-mode_ powers through `sudo`.

- **Add a user to sudo/wheel group:**
    
    text
    
    ```
    sudo usermod -aG sudo alice
    ```
    
    (Debian/Ubuntu distributions use `sudo` group. On Red Hat/CentOS, it’s often `wheel`.)
    
- **Verify sudo access:**  
    Switch to Alice, then run:
    
    text
    
    ```
    sudo whoami
    ```
    
    → if it says `root`, she has ascended. 👑
    
- **Modify sudoers file directly:**
    
    text
    
    ```
    sudo visudo
    ```
    
    - Safest way to edit `/etc/sudoers`.
    - Lets you tightly control access: e.g. allow Alice to only restart nginx, but not blow up the whole system.

---
## 🔄 Switching Users

- **Switch to another user’s shell:**
    
    text
    
    ```
    su - bob
    ```
    
    → "become Bob," full environment and all.
    
- **Temporarily run commands as another user:**
    
    text
    
    ```
    sudo -u bob command_here
    ```
    
    → Useful when you want just one action done as Bob, then return to yourself.
    
- **Go big and become root:**
    
    - text
    
    ```
    sudo -i
    ```
    
    or
    
    text
    
    ```
    su -
    ```
    
    (if you know the root password). Now you _are_ the system.
    

---
# 🧩 Hacker Practice Mission

1. Create a group `hackers`.
    
    text
    
    ```
    sudo groupadd hackers
    ```
    
2. Add two users, `neo` and `trinity`.
    
    text
    
    ```
    sudo adduser neo
    sudo adduser trinity
    ```
    
3. Add them both to `hackers` group.
    
    1. text
    
    ```
    sudo usermod -aG hackers neo
    sudo usermod -aG hackers trinity
    ```
    
4. Make `neo` a sudoer (leader of the crew).
    
    text
    
    ```
    sudo usermod -aG sudo neo
    ```
    
5. Switch to Trinity and confirm she’s in group `hackers` but not sudo.
    
    1. text
    
    ```
    su - trinity
    groups
    ```
    
2. Switch to Neo, run `sudo whoami`, and smile as you wield root powers.

---

## 🧠 Key Takeaway

- **Users** = individuals.
- **Groups** = teams.
- **sudo** = god‑mode access.
- **su/sudo -u** = shapeshifting into another identity.

It’s like running a hacker guild: some members are foot soldiers, some are team captains, and one or two get crowned as Archmage Root. 🔮

---
