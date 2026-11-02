# INTRODUCTION TO CYBER SECURITY LAB

PRACTICAL FILE – ALL 10 EXPERIMENTS

Student Details

- Name: ______________________
    
- Roll Number: ______________________
    
- Enrollment Number: ______________________
    
- Class / Semester: ______________________
    
- Subject: Introduction to Cyber Security Lab
    
- Submitted To: ______________________
    
- Academic Year: ______________________
    

# Practical 1: Study of Cyber Security

Aim: To identify common cyber threats, vulnerabilities, and basic security measures.

Theory:

Cyber Security is the practice of protecting computers, networks, applications, and data from unauthorized access, attacks, and damage.

Common Cyber Threats:

1. Virus: Malicious software that infects files.
    
2. Worm: Malware that spreads across networks.
    
3. Trojan Horse: Malicious software disguised as legitimate software.
    
4. Phishing: Fraudulent messages designed to steal sensitive information.
    
5. Ransomware: Malware that locks or encrypts files and demands payment.
    
6. Spyware: Software that secretly collects user information.
    

Common Vulnerabilities:

- Weak passwords.
    
- Outdated software.
    
- Unsecured Wi-Fi networks.
    
- Improper access permissions.
    
- Lack of data encryption.
    

Basic Security Measures:

- Use strong passwords.
    
- Enable two-factor authentication.
    
- Install antivirus software.
    
- Update operating systems regularly.
    
- Use firewalls.
    
- Take regular data backups.
    

Procedure:

1. Study different types of cyber threats.
    
2. Identify common system vulnerabilities.
    
3. Understand the importance of antivirus and firewalls.
    
4. Study preventive security measures.
    

Result: Successfully studied common cyber threats, vulnerabilities, and basic cyber security measures.

# Practical 2: Password and Authentication

Aim: To create strong passwords and study different authentication methods.

Theory:

Authentication is the process of verifying the identity of a user before allowing access to a system.

Types of Authentication:

1. Password-based authentication.
    
2. Biometric authentication.
    
3. OTP-based authentication.
    
4. Two-factor authentication (2FA).
    
5. Multi-factor authentication (MFA).
    

Characteristics of a Strong Password:

- Minimum 12 characters.
    
- Contains uppercase and lowercase letters.
    
- Includes numbers and special characters.
    
- Avoids personal information.
    
- Should be unique for every account.
    

Python Program: Password Strength Checker

python

```
import re

password = input("Enter password: ")

if (len(password) >= 12 and
    re.search(r"[A-Z]", password) and
    re.search(r"[a-z]", password) and
    re.search(r"[0-9]", password) and
    re.search(r"[^A-Za-z0-9]", password)):

    print("Strong Password")
else:
    print("Weak Password")
```

Sample Output:

text

```
Enter password: Hello@123456
Strong Password
```

Procedure:

1. Open Python IDLE or any Python editor.
    
2. Write the above program.
    
3. Execute the program.
    
4. Enter a password.
    
5. Observe the password strength result.
    

Result: Successfully created a password strength checker and studied different authentication methods.

# Practical 3: Basic Cryptography

Aim: To implement simple encryption and decryption techniques using Caesar Cipher and substitution techniques.

Theory:

Cryptography is the technique of protecting information by converting readable data (plaintext) into an unreadable format (ciphertext).

- Encryption: Converting plaintext into ciphertext.
    
- Decryption: Converting ciphertext into plaintext.
    
- Key: Information used to encrypt or decrypt data.
    

A. Caesar Cipher

Caesar Cipher is a substitution encryption technique in which every letter is shifted by a fixed number of positions.

Python Program:

python

```
def encrypt(text, shift):
    result = ""

    for char in text:
        if char.isalpha():
            base = ord('A') if char.isupper() else ord('a')
            result += chr((ord(char) - base + shift) % 26 + base)
        else:
            result += char

    return result


def decrypt(text, shift):
    return encrypt(text, -shift)


message = input("Enter message: ")
key = int(input("Enter shift key: "))

encrypted = encrypt(message, key)

print("Encrypted:", encrypted)
print("Decrypted:", decrypt(encrypted, key))
```

Sample Output:

text

```
Enter message: HELLO
Enter shift key: 3

Encrypted: KHOOR
Decrypted: HELLO
```

B. Substitution Cipher

A substitution cipher replaces each letter with another letter according to a fixed mapping.

Example:

text

```
Plaintext:  ABCDEFGHIJKLMNOPQRSTUVWXYZ
Ciphertext: QWERTYUIOPASDFGHJKLZXCVBNM
```

Procedure:

1. Enter a message.
    
2. Select an encryption key.
    
3. Apply the Caesar Cipher.
    
4. Generate encrypted text.
    
5. Decrypt the ciphertext.
    
6. Verify the original message.
    

Result: Successfully implemented Caesar Cipher encryption and decryption and studied substitution cipher techniques.

Note: These classical ciphers are educational and are not secure for real-world communication.

# Practical 4: Hashing

Aim: To generate and verify MD5 and SHA-256 hash values to understand data integrity.

Theory:

Hashing is the process of converting input data into a fixed-length string called a hash value.

Hashing is commonly used for:

- Data integrity verification.
    
- Password storage (with appropriate password-hashing algorithms).
    
- Digital signatures.
    
- File verification.
    

Types of Hashing:

1. MD5 – Produces a 128-bit hash.
    
2. SHA-256 – Produces a 256-bit hash.
    

Python Program:

python

```
import hashlib

text = input("Enter text: ")

md5_hash = hashlib.md5(
    text.encode()
).hexdigest()

sha256_hash = hashlib.sha256(
    text.encode()
).hexdigest()

print("MD5 Hash:", md5_hash)
print("SHA-256 Hash:", sha256_hash)
```

Sample Output:

text

```
Enter text: Hello

MD5 Hash:
8b1a9953c4611296a827abf8c47804d7

SHA-256 Hash:
185f8db32271fe25f561a6fc938b2e26
```

The SHA-256 output above is shortened for display; the actual hash contains 64 hexadecimal characters.

Procedure:

1. Open Python IDLE.
    
2. Import the hashlib library.
    
3. Enter the text.
    
4. Generate MD5 and SHA-256 hashes.
    
5. Compare the hash values.
    

Result: Successfully generated MD5 and SHA-256 hash values and understood the importance of data integrity.

# Practical 5: RSA Cryptography

Aim: To understand and implement basic RSA encryption and decryption.

Theory:

RSA is an asymmetric cryptographic algorithm that uses two keys:

- Public Key: Used for encryption.
    
- Private Key: Used for decryption.
    

RSA is based on mathematical properties of large prime numbers.

RSA Algorithm Steps:

1. Select two prime numbers, p and q.
    
2. Calculate n = p × q.
    
3. Calculate φ(n) = (p − 1) × (q − 1).
    
4. Select a public exponent e.
    
5. Calculate private exponent d.
    
6. Encrypt the message.
    
7. Decrypt the encrypted message.
    

Python Program (Educational Example):

python

```
# Small numbers for learning only

p = 3
q = 11

n = p * q
phi = (p - 1) * (q - 1)

e = 3
d = 7

message = 4

# Encryption
cipher = pow(message, e, n)

# Decryption
plain = pow(cipher, d, n)

print("Original Message:", message)
print("Encrypted Message:", cipher)
print("Decrypted Message:", plain)
```

Sample Output:

text

```
Original Message: 4
Encrypted Message: 64
Decrypted Message: 4
```

Procedure:

1. Select two prime numbers.
    
2. Calculate n and φ(n).
    
3. Generate public and private exponents.
    
4. Encrypt the message.
    
5. Decrypt the ciphertext.
    
6. Verify the original message.
    

Result: Successfully understood the working of RSA encryption and decryption.

Note: This is a tiny mathematical demonstration, not secure RSA. Real systems require appropriately sized keys and secure padding such as OAEP.

# Practical 6: Network Security Using Wireshark

Aim: To use Wireshark to capture and analyze basic network packets.

Theory:

Wireshark is a network protocol analyzer used to capture and examine network traffic.

It helps network administrators understand communication between devices and troubleshoot network problems.

Features of Wireshark:

- Captures network packets.
    
- Displays source and destination addresses.
    
- Identifies network protocols.
    
- Analyzes packet information.
    
- Helps troubleshoot network issues.
    

Requirements:

- Computer or laptop.
    
- Wireshark software.
    
- Internet or local network connection.
    

Procedure:

1. Download and install Wireshark from its official website.
    
2. Open Wireshark.
    
3. Select your active network interface.
    
4. Start packet capture.
    
5. Generate normal traffic by visiting a website you own or are authorized to access.
    
6. Observe the captured packets.
    
7. Examine protocol names such as TCP, DNS, UDP, and TLS.
    
8. Stop the capture.
    
9. Save the capture file for analysis.
    

Example Display Filter:

text

```
dns
```

This displays DNS packets.

Other useful filters:

text

```
tcp
udp
icmp
tls
```

Observation Table:

|Protocol|Purpose|
|---|---|
|TCP|Reliable data communication|
|UDP|Fast data transmission|
|DNS|Domain name resolution|
|ICMP|Network diagnostics|
|TLS|Secure communication|

Result: Successfully used Wireshark to capture and study basic network packets and protocols.

Safety: Capture only your own traffic or traffic you are explicitly authorized to inspect.

# Practical 7: Firewall

Aim: To study and configure basic firewall rules for network security.

Theory:

A firewall is a security system that monitors incoming and outgoing network traffic and allows or blocks traffic according to predefined rules.

Types of Firewalls:

1. Packet Filtering Firewall.
    
2. Stateful Inspection Firewall.
    
3. Proxy Firewall.
    
4. Next-Generation Firewall.
    

Functions of a Firewall:

- Blocks unauthorized access.
    
- Controls network traffic.
    
- Protects internal networks.
    
- Monitors incoming and outgoing connections.
    
- Helps prevent unauthorized communication.
    

Procedure (Windows):

1. Open Control Panel.
    
2. Select System and Security.
    
3. Open Windows Defender Firewall.
    
4. Check whether the firewall is enabled.
    
5. Select Advanced Settings.
    
6. Study inbound and outbound rules.
    
7. Examine existing rules without changing essential system settings.
    

[metablaster.github.io](https://metablaster.github.io/WindowsFirewallRuleset/MonitoringFirewall.html?utm_source=chatgpt.com)

Basic Firewall Rule Examples:

|Rule|Action|
|---|---|
|Allow trusted application|Allow|
|Block unauthorized application|Block|
|Allow required network service|Allow|
|Block suspicious incoming traffic|Block|

Linux Firewall Observation:

The following commands can be used to check firewall status:

bash

```
sudo ufw status
```

To display detailed status:

bash

```
sudo ufw status verbose
```

Result: Successfully studied firewall configuration, inbound and outbound rules, and the importance of firewalls in network security.

# Practical 8: Web Security – SQL Injection and XSS

Aim: To study basic web security issues such as SQL Injection and Cross-Site Scripting (XSS) in a controlled lab environment.

Theory:

Web security protects websites and web applications from unauthorized access, data theft, and malicious activities.

A. SQL Injection

SQL Injection is a vulnerability that occurs when an application improperly handles user input in database queries.

Possible Effects:

- Unauthorized database access.
    
- Exposure of confidential information.
    
- Modification of database records.
    
- Authentication bypass.
    

Prevention:

- Use parameterized SQL queries.
    
- Validate user input.
    
- Apply least-privilege database permissions.
    
- Avoid building SQL statements by directly joining user input.
    

Secure Python Example:

python

```
import sqlite3

connection = sqlite3.connect(":memory:")

connection.execute(
    "CREATE TABLE users (username TEXT)"
)

connection.execute(
    "INSERT INTO users VALUES (?)",
    ("student",)
)

username = input("Enter username: ")

result = connection.execute(
    "SELECT * FROM users WHERE username = ?",
    (username,)
)

print(result.fetchall())

connection.close()
```

This example uses parameterized queries to handle user input safely.

B. Cross-Site Scripting (XSS)

XSS is a web security vulnerability in which untrusted content is incorrectly handled by a web application and may execute as browser script.

Types of XSS:

1. Stored XSS.
    
2. Reflected XSS.
    
3. DOM-based XSS.
    

Prevention:

- Encode untrusted output.
    
- Validate user input.
    
- Use Content Security Policy (CSP).
    
- Avoid inserting untrusted content as HTML.
    
- Use secure web development frameworks.
    

Secure HTML Example:

HTML

Wrap linesCopy codePreview

```
<!DOCTYPE html>
<html>
<body>

<h2>Safe User Input</h2>

<input id="username" placeholder="Enter name">

<button onclick="showName()">Submit</button>

<p id="output"></p>

<script>
function showName() {
    let name = document.getElementById("username").value;

    document.getElementById("output").textContent = name;
}
</script>

</body>
</html>
```

The `textContent` property displays user input as text rather than interpreting it as HTML.

Procedure:

1. Study SQL Injection and XSS vulnerabilities.
    
2. Understand how unsafe input handling creates security risks.
    
3. Study parameterized SQL queries.
    
4. Examine safe output encoding.
    
5. Understand preventive security techniques.
    

Result: Successfully studied SQL Injection and XSS vulnerabilities and their preventive security measures.

# Practical 9: Operating System Security

Aim: To study user accounts, file permissions, access control, and basic security settings in Windows/Linux.

Theory:

Operating System Security protects system resources, files, applications, and user accounts from unauthorized access.

Important Concepts:

1. User Accounts: Different users can have separate accounts.
    
2. File Permissions: Control who can read, write, or execute files.
    
3. Access Control: Determines which resources users can access.
    
4. Authentication: Verifies user identity.
    
5. Updates: Fix security vulnerabilities.
    

Windows Security Procedure:

1. Open Settings.
    
2. Navigate to Accounts.
    
3. Study user account settings.
    
4. Open Windows Security.
    
5. Check virus and threat protection.
    
6. Examine firewall settings.
    
7. Review available security updates.
    

[techradar.com](https://www.techradar.com/reviews/windows-defender?utm_source=chatgpt.com)

Linux Commands:

Check current username:

bash

```
whoami
```

Display file permissions:

bash

```
ls -l
```

Display current user information:

bash

```
id
```

Check system information:

bash

```
uname -a
```

Example of Linux Permissions:

text

```
-rwxr-xr--
```

Meaning:

- Owner: Read, Write, Execute.
    
- Group: Read, Execute.
    
- Others: Read only.
    

Procedure:

1. Open Windows or Linux.
    
2. Check existing user accounts.
    
3. Examine file permissions.
    
4. Study access control.
    
5. Check security updates.
    
6. Observe antivirus and firewall settings.
    

Result: Successfully studied user accounts, file permissions, access control, and basic operating system security.

# Practical 10: Cyber Security Incident Report

Aim: To study a simple cyber security incident and prepare a basic incident report covering the threat, impact, and preventive measures.

Theory:

A cyber security incident is an event that threatens the confidentiality, integrity, or availability of computer systems and data.

Common Cyber Security Incidents:

- Phishing attacks.
    
- Malware infections.
    
- Ransomware attacks.
    
- Unauthorized access.
    
- Data breaches.
    
- Password theft.
    

Sample Cyber Security Incident Report

## Cyber Security Incident Report

SAMPLE CASE STUDY

1. Incident Title

Phishing Email Attack.

2. Incident Description

An employee received a suspicious email claiming to be from a banking organization. The email requested the employee to verify account information through a link.

3. Type of Threat

Phishing Attack.

4. Cause of Incident

- Suspicious email.
    
- Misleading website link.
    
- Lack of user awareness.
    
- Social engineering techniques.
    

5. Impact of Incident

- Possible theft of login credentials.
    
- Risk of unauthorized account access.
    
- Possible exposure of confidential information.
    

6. Preventive Measures

- Avoid clicking suspicious links.
    
- Enable multi-factor authentication.
    
- Use email filtering systems.
    
- Provide cyber security awareness training.
    
- Change compromised passwords.
    
- Monitor accounts for suspicious activity.
    

7. Response and Recovery

- Report the suspicious email to the IT department.
    
- Isolate affected systems if necessary.
    
- Reset compromised credentials.
    
- Review account activity.
    
- Document the incident and take preventive action.
    

8. Conclusion

Phishing attacks can cause serious security problems. Proper awareness, authentication, and email security can help prevent such incidents.

Procedure:

1. Select a cyber security incident.
    
2. Study the nature of the threat.
    
3. Identify the cause of the incident.
    
4. Analyze its possible impact.
    
5. Suggest preventive measures.
    
6. Prepare an incident report.
    

Result: Successfully studied a cyber security incident and prepared an incident report covering the threat, impact, response, and preventive measures.

# Overall Conclusion

Through these ten practical exercises, we studied the fundamental concepts of cyber security, password authentication, cryptography, hashing, RSA encryption, network security, firewalls, web security, operating system security, and cyber security incident management.

We also learned how to use basic security tools and Python programs to understand security concepts and protect computer systems against common cyber threats.

Final Result: Successfully completed all ten practical exercises of the Introduction to Cyber Security Lab.

## Important: Practical File Submission Checklist

Your submission progress

## 0/12

Cover page and student details

Practical 1 – Cyber Security

Practical 2 – Password and Authentication

Practical 3 – Basic Cryptography

Practical 4 – Hashing

Practical 5 – RSA Cryptography

Practical 6 – Wireshark

Practical 7 – Firewall

Practical 8 – Web Security

Practical 9 – Operating System Security

Practical 10 – Incident Report

Overall conclusion

My suggestion: Write each practical with its aim, theory, procedure, program, output, and result. Attach screenshots of your actual program executions wherever required by your teacher.

You can use this as your first draft and add your name, roll number, and actual lab observations before submission.