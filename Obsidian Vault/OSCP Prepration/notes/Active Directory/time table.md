### Day 1 — AD Core Concepts

**Goal:** Understand what you're attacking before attacking it.

- Domains, forests, trees, trusts (one-way vs two-way, transitive)
- Domain Controllers, FSMO roles, Global Catalog
- Organizational Units, Group Policy Objects (GPOs)
- Users, groups (security vs distribution), computer objects
- SYSVOL, NTDS.dit basics
- **Lab:** Spin up a Windows Server domain controller (eval ISO) + join 1-2 client VMs. Use Windows Server 2019/2022 trial.

### Day 2 — LDAP Deep Dive

**Goal:** Understand how AD stores and exposes data.

- LDAP protocol basics, distinguished names (DN), object classes, attributes
- LDAP queries — filters, `ldapsearch`, PowerShell `Get-ADUser`/`Get-ADObject`
- ADSI, ADSI Edit
- Tools: `ldapdomaindump`, BloodHound's data collection (SharpHound) relies heavily on this
- **Lab:** Run LDAP queries against your lab DC, enumerate users/groups/OUs manually and via PowerShell.

### Day 3 — Kerberos Fundamentals

**Goal:** This is the conceptual core of the week — don't rush it.

- AS-REQ/AS-REP, TGT, KDC
- TGS-REQ/TGS-REP, service tickets
- Ticket structure, encryption types (RC4 vs AES), SPNs
- KRBTGT account, ticket lifetime, PAC
- **Resource:** Diagram the full auth flow yourself — draw it until you can explain it without notes.
- **Lab:** Capture Kerberos traffic with Wireshark during a normal login; identify AS-REQ/TGS-REQ packets.

### Day 4 — Kerberoasting & AS-REP Roasting

**Goal:** First real attack, tied directly to Day 3 concepts.

- Kerberoasting: requesting TGS for accounts with SPNs, cracking offline (service account weak passwords)
- Tools: Rubeus, Impacket's `GetUserSPNs.py`, hashcat mode 13100
- AS-REP Roasting: accounts with "do not require Kerberos preauth"
- **Lab:** Create a service account with an SPN and weak password in your domain, Kerberoast it, crack with hashcat.

### Day 5 — Pass-the-Hash & Credential Attacks

**Goal:** Shift from Kerberos to NTLM-based lateral movement.

- NTLM auth basics (brief, for contrast with Kerberos)
- Pass-the-Hash (PtH): using NTLM hash without cracking password
- Mimikatz — `sekurlsa::logonpasswords`, `sekurlsa::pth`
- Overpass-the-Hash (NTLM hash → Kerberos TGT)
- Tools: Impacket `psexec.py`/`wmiexec.py` with `-hashes`, CrackMapExec/NetExec
- **Lab:** Dump hashes with Mimikatz on a compromised lab machine, use PtH to authenticate to another machine.

### Day 6 — Domain Trusts & Privilege Escalation Paths

**Goal:** Tie it together with attack-path thinking.

- Domain/forest trust exploitation, SID history abuse, trust tickets
- DCSync attack (mimikatz `lsadump::dcsync`) — abusing replication rights
- Golden Ticket / Silver Ticket (forging TGTs/service tickets with KRBTGT hash)
- Constrained/unconstrained delegation abuse
- **Lab:** Run BloodHound (SharpHound collector) against your lab domain, map attack paths, identify a shortest path to Domain Admin.

### Day 7 — Integration, BloodHound Mastery & Review

**Goal:** Consolidate everything into a coherent attack chain.

- Full BloodHound analysis: Cypher queries, custom queries for Kerberoastable/AS-REP roastable users, unconstrained delegation
- Chain a full attack: enumerate → Kerberoast → crack → lateral move (PtH) → DCSync → Golden Ticket
- Review notes, redo the hardest lab from the week
- Write a one-page summary of the full AD kill chain in your own words