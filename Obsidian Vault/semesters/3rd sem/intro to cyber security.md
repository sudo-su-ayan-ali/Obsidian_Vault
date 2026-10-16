# UNIT I — Introduction to Cyber Security

## A. Short Answer Questions

### 1. What is Cyber Security?

Cyber security is the practice of protecting computers, networks, applications, and data from unauthorized access, cyberattacks, damage, or theft.

Example: Using a firewall and antivirus software to protect a computer from malware.

### 2. What is a security vulnerability?

A security vulnerability is a weakness in a system, application, network, or configuration that an attacker can exploit to compromise security.

Example: Outdated software containing a known security flaw.

### 3. What is authentication?

Authentication is the process of verifying the identity of a user, device, or system before granting access.

Example: Logging into an account using a username and password or fingerprint.

### 4. What is a browser attack?

A browser attack is a cyberattack that exploits weaknesses in a web browser, its extensions, or the websites it visits to steal information, execute malicious code, or compromise a user's security.

Example: A malicious website exploiting an outdated browser to run unauthorized code.

## B. Medium Answer Questions

### 1. Explain computer security, threats, vulnerabilities and controls with suitable examples.

Computer security is the protection of computer systems, software, hardware, and data against unauthorized access, modification, destruction, or disruption.

Security threats: Threats are potential causes of harm to a computer system.

Examples:

- Malware can damage files or steal information.
    
- Phishing can trick users into revealing passwords.
    
- An insider may misuse authorized access.
    

Security vulnerabilities: Vulnerabilities are weaknesses that threats can exploit.

Examples:

- Weak passwords.
    
- Outdated operating systems.
    
- Incorrect file permissions.
    
- Misconfigured servers.
    

Security controls: Controls are safeguards used to prevent, detect, or reduce security risks.

|Control type|Examples|
|---|---|
|Preventive|Firewalls, strong passwords, access restrictions|
|Detective|Log monitoring, intrusion detection systems|
|Corrective|Patching, restoring backups, removing malware|
|Physical|Locks, CCTV, biometric door access|

Conclusion: Computer security requires identifying vulnerabilities, understanding threats, and implementing appropriate controls to protect systems and data.

### 2. Explain authentication, access control and cryptography in cyber security.

These three mechanisms help protect systems, users, and sensitive information.

1. Authentication

Authentication verifies who a user or system claims to be.

Common methods include:

- Passwords and PINs.
    
- Fingerprints and facial recognition.
    
- Security tokens.
    
- Multi-factor authentication (MFA).
    

Example: A user enters a password and an OTP to log in to a banking application.

2. Access control

Access control determines which resources an authenticated user is allowed to access and what actions they can perform.

Common models include:

- DAC: The resource owner controls permissions.
    
- MAC: Access is based on centrally defined security classifications and rules.
    
- RBAC: Permissions are assigned according to roles.
    
- ABAC: Access depends on attributes such as user, resource, and context.
    

Example: A student can view marks, but only an authorized teacher can modify them.

3. Cryptography

Cryptography protects information using mathematical techniques.

Its main applications include:

- Encryption: Converts readable data into ciphertext to protect confidentiality.
    
- Hashing: Produces a fixed-length digest used for integrity verification and other security purposes.
    
- Digital signatures: Help verify authenticity and detect unauthorized changes.
    
- Key exchange: Allows parties to establish shared cryptographic keys.
    

Example: HTTPS uses cryptographic protocols to protect data exchanged between a browser and a website.

Conclusion: Authentication verifies identity, access control enforces permissions, and cryptography protects information and supports secure communication.

### 3. Describe common web attacks that target users and explain how attackers obtain user or website data.

Web attacks exploit weaknesses in websites, browsers, applications, or user behavior to steal information or gain unauthorized access.

Common attacks:

1. Phishing: Attackers create fake login pages or messages to trick users into revealing credentials.
    
2. Cross-Site Scripting (XSS): Malicious scripts execute in a victim's browser through a vulnerable website.
    
3. SQL Injection (SQLi): Improperly handled database inputs allow attackers to interfere with database queries.
    
4. Cross-Site Request Forgery (CSRF): Tricks a logged-in user's browser into submitting an unwanted request.
    
5. Session hijacking: An attacker steals or misuses a session token to impersonate a user.
    
6. Brute-force attacks: Repeated login attempts are used to guess weak passwords.
    
7. Malware attacks: Malicious software steals files, credentials, or other sensitive information.
    

How attackers obtain data:

- Stealing usernames, passwords, and session tokens.
    
- Exploiting application vulnerabilities.
    
- Accessing exposed databases or misconfigured cloud storage.
    
- Tricking employees into disclosing confidential information.
    
- Intercepting data when communications are not adequately protected.
    
- Using malware to collect information from compromised devices.
    

Prevention measures:

- Validate and safely handle user input.
    
- Use parameterized SQL queries.
    
- Implement MFA and strong session management.
    
- Keep software updated.
    
- Use HTTPS, secure cookies, and appropriate security headers.
    
- Train users to recognize phishing attempts.
    

### 4. Explain different types of email attacks and discuss basic measures for protecting users from them.

Email attacks use email messages or attachments to steal information, distribute malware, or gain unauthorized access.

Types of email attacks:

1. Phishing: Fraudulent emails designed to steal passwords or financial information.
    
2. Spear phishing: Targeted phishing aimed at a specific person or organization.
    
3. Business Email Compromise (BEC): An attacker impersonates a trusted person, such as a manager or supplier, to request money or sensitive information.
    
4. Email spoofing: Forging email sender information to make a message appear to come from a trusted source.
    
5. Malicious attachments: Files containing malware that infect a device when opened or executed.
    
6. Malicious links: Links that direct users to fraudulent websites or trigger harmful actions.
    
7. Spam: Unwanted bulk messages, sometimes used to distribute scams or malware.
    

Protection measures:

- Verify the sender's identity before responding to unusual requests.
    
- Avoid suspicious links and unexpected attachments.
    
- Enable multi-factor authentication.
    
- Use updated antivirus and email security filters.
    
- Implement SPF, DKIM, and DMARC to help protect email domains against spoofing.
    
- Confirm financial transfers or sensitive requests through a separate trusted communication channel.
    
- Report suspicious messages to the security team.
    
- Keep operating systems and applications updated.
    

Conclusion: User awareness, email filtering, identity verification, and strong authentication significantly reduce email-related risks.

## C. Long Answer Questions

### 1. Explain the fundamentals of cyber security in detail. Discuss computer security, threats, harm, vulnerabilities, security controls, authentication, access control and cryptography.

### Introduction

Cyber security is the practice of protecting computer systems, networks, applications, and information from cyber threats, unauthorized access, damage, and theft. It helps maintain the confidentiality, integrity, and availability of information.

### 1. Computer Security

Computer security protects computer hardware, software, operating systems, networks, and stored data against unauthorized activities.

Its main objectives are:

- Confidentiality: Information is accessible only to authorized users.
    
- Integrity: Information remains accurate and is not modified without authorization.
    
- Availability: Systems and data remain accessible when required.
    

These three objectives are known as the CIA triad.

Example: A banking system uses access permissions to protect customer information, transaction validation to maintain integrity, and backups to support availability.

### 2. Security Threats

A security threat is a potential cause of an unwanted security incident.

Common threats include:

- Malware: Malicious software such as viruses, worms, ransomware, and spyware.
    
- Phishing: Fraudulent messages used to steal sensitive information.
    
- Insider threats: Employees or authorized users misusing their access.
    
- Denial-of-Service (DoS): Attempts to make a service unavailable.
    
- Password attacks: Attempts to guess, steal, or misuse passwords.
    
- Social engineering: Manipulating people into revealing information or performing unsafe actions.
    

### 3. Harm Caused by Cyberattacks

Cyberattacks can cause different types of harm:

- Financial loss: Theft, fraud, ransom payments, or recovery expenses.
    
- Data loss: Deletion, corruption, or theft of information.
    
- Privacy violations: Exposure of personal or confidential information.
    
- Operational disruption: Systems or services becoming unavailable.
    
- Reputational damage: Loss of customer trust.
    
- Legal consequences: Penalties or liability resulting from security failures.
    

For example, ransomware can encrypt an organization's files, interrupt operations, and create expensive recovery work.

### 4. Security Vulnerabilities

A vulnerability is a weakness that an attacker may exploit to compromise a system.

Common vulnerabilities include:

- Weak or reused passwords.
    
- Outdated software.
    
- Improper input validation.
    
- Incorrect access permissions.
    
- Misconfigured servers.
    
- Unnecessary open ports and services.
    
- Insecure application programming interfaces (APIs).
    

Example: An application using unsafe SQL queries may be vulnerable to SQL injection.

Organizations identify vulnerabilities through vulnerability scanning, secure code reviews, penetration testing, and configuration audits.

### 5. Security Controls

Security controls are safeguards used to prevent, detect, or correct security problems.

|Type|Purpose|Example|
|---|---|---|
|Preventive|Stop incidents before they occur|MFA, firewalls|
|Detective|Identify suspicious activity|Log monitoring, IDS|
|Corrective|Recover from or fix incidents|Patching, malware removal|
|Physical|Protect facilities and devices|Locks, CCTV|
|Administrative|Establish security rules|Security policies, training|

A strong security program combines multiple controls rather than relying on one technology.

### 6. Authentication

Authentication verifies the identity of a user, device, or system before access is granted.

![](https://www.google.com/s2/favicons?domain=https://csrc.nist.gov&sz=32)

CSRC

+1

Common authentication factors are:

1. Something you know: Password or PIN.
    
2. Something you have: Security key or authenticator device.
    
3. Something you are: Fingerprint or facial characteristics.
    

Multi-factor authentication (MFA) combines two or more distinct authentication factors.

Example: A user signs in using a password and a hardware security key.

### 7. Access Control

Access control determines what an authenticated user can access and which actions they can perform.

Common access control models include:

- Discretionary Access Control (DAC): The resource owner determines permissions.
    
- Mandatory Access Control (MAC): A central policy determines access based on security classifications.
    
- Role-Based Access Control (RBAC): Permissions are assigned according to job roles.
    
- Attribute-Based Access Control (ABAC): Access depends on attributes and contextual rules.
    

The principle of least privilege means users should receive only the permissions necessary to perform their duties.

Example: A university student can view their results, while authorized faculty members can enter or update marks.

### 8. Cryptography

Cryptography uses mathematical techniques to protect information and communications.

Its important applications are:

- Encryption: Protects confidentiality by converting plaintext into ciphertext.
    
- Decryption: Converts ciphertext back into readable information using the appropriate key.
    
- Hashing: Produces a digest that can help detect changes to data.
    
- Digital signatures: Support verification of message authenticity and integrity.
    
- Key exchange: Enables parties to establish cryptographic keys securely.
    

Cryptography includes symmetric encryption, asymmetric cryptography, hashing algorithms, and digital signature schemes.

Example: HTTPS uses TLS to protect communication between a browser and a website.

### 9. Measures to Improve Cyber Security

Organizations and individuals should:

- Install security updates regularly.
    
- Use strong, unique passwords and MFA.
    
- Configure firewalls and access permissions.
    
- Encrypt sensitive information.
    
- Maintain tested backups.
    
- Monitor system logs for suspicious activities.
    
- Train users to identify phishing and social engineering.
    
- Conduct regular vulnerability assessments.
    
- Maintain an incident response and recovery plan.
    

### Conclusion

Cyber security is essential for protecting information, systems, and services. Effective security requires identifying threats and vulnerabilities, understanding potential harm, and implementing suitable controls. Authentication, access control, cryptography, monitoring, and regular updates work together to reduce security risks.

### 2. Explain web and user-side security threats in detail. Discuss browser attacks, web attacks, attacks targeting users, methods of obtaining user or website data and email attacks.

### Introduction

Web and user-side security focuses on protecting web browsers, websites, web applications, user devices, accounts, and communications. Attackers exploit software weaknesses and human errors to steal information, gain unauthorized access, or disrupt services.

### 1. Browser Attacks

A browser attack targets weaknesses in a browser, its extensions, or the content it processes.

Common browser attacks include:

a. Malicious websites: A website may attempt to trick users into downloading malware, entering credentials, or granting unwanted permissions.

b. Cross-Site Scripting (XSS): A vulnerable website allows malicious JavaScript to execute in a visitor's browser.

c. Malicious browser extensions: An extension may collect browsing information, access sensitive page content, or modify websites beyond what is necessary.

d. Drive-by downloads: Visiting a compromised website can trigger an unwanted download or, in certain circumstances, exploit a browser or plugin vulnerability.

e. Clickjacking: A malicious page tricks a user into clicking a hidden or disguised interface element.

Prevention: Keep browsers updated, install extensions only from trusted sources, review permissions, avoid suspicious downloads, and use appropriate browser security settings.

### 2. Web Attacks

Web attacks target vulnerabilities in websites, web applications, APIs, and their supporting infrastructure.

|Attack|Explanation|
|---|---|
|SQL Injection|Unsafe database queries allow attacker-controlled input to interfere with database operations.|
|Cross-Site Scripting (XSS)|Malicious scripts execute in a victim's browser through a vulnerable application.|
|Cross-Site Request Forgery (CSRF)|A user's browser is tricked into making an unwanted authenticated request.|
|Broken access control|Users access resources or perform actions beyond their permissions.|
|Directory traversal|Improper file-path handling allows access to files outside the intended directory.|
|Server-Side Request Forgery (SSRF)|An attacker manipulates a server into making unintended requests to other resources.|
|Denial-of-Service (DoS)|An application or service is overwhelmed or otherwise made unavailable.|

Example: If an online store fails to enforce authorization on an order endpoint, one customer may be able to view another customer's order.

Prevention measures:

- Validate input and encode output appropriately.
    
- Use parameterized database queries.
    
- Enforce authorization on the server for every sensitive operation.
    
- Apply secure session management.
    
- Configure security headers and HTTPS.
    
- Keep application frameworks and dependencies updated.
    
- Conduct authorized security testing.
    

### 3. Attacks Targeting Users

These attacks exploit human behavior, trust, or weak account security.

a. Social engineering: Manipulating a person into revealing information or performing an unsafe action.

b. Phishing: Sending fraudulent messages to steal credentials or financial information.

c. Credential stuffing: Using previously exposed username-password combinations against other services.

d. Brute-force attacks: Trying many possible passwords to gain access.

e. Identity theft: Misusing personal information to impersonate another person.

f. Malware infection: Tricking a user into running malicious software that steals information or damages a device.

Example: A user receives a fake bank message and enters their login details on a fraudulent website.

Prevention: Use MFA, unique passwords, trusted password managers, software updates, and independent verification of unusual requests.

### 4. Methods of Obtaining User or Website Data

Attackers may obtain data through several methods:

1. Credential theft: Phishing, password reuse, malware, or exposed credentials can provide access to user accounts.

2. Application vulnerabilities: SQL injection, broken access control, and insecure APIs can expose records that should remain protected.

3. Session theft: Stolen session tokens may allow an attacker to impersonate an authenticated user.

4. Misconfigured services: Public databases, storage buckets, or administrative interfaces may expose confidential data.

5. Network interception: Poorly protected communications can expose information in transit. Properly configured TLS helps protect against this risk.

6. Malware and spyware: Compromised devices may reveal files, keystrokes, credentials, or browsing activity.

7. Social engineering: Attackers may persuade employees or users to disclose confidential information.

8. Public information gathering: Websites, public records, and exposed repositories may reveal information that helps an attacker prepare a targeted attack.

Protection measures: Restrict access to sensitive data, encrypt data at rest and in transit, patch vulnerabilities, secure APIs, monitor logs, and minimize the collection and retention of personal information.

### 5. Email Attacks

Email is commonly used to deliver fraudulent messages, steal credentials, and distribute malicious software.

a. Phishing: Fake emails imitate banks, services, or organizations to steal information.

b. Spear phishing: Personalized messages target a particular individual or organization.

c. Business Email Compromise (BEC): An attacker impersonates an executive, employee, or supplier to manipulate financial transactions or obtain confidential data.

d. Email spoofing: Forged sender information makes an email appear to originate from another address or domain.

e. Malicious attachments: Documents or executable files may contain malware or exploit vulnerabilities.

f. Malicious links: Links direct recipients to fraudulent websites or harmful downloads.

g. Spam: Unwanted bulk messages may contain scams, advertisements, or malicious content.

### 6. Email Security Measures

Organizations and users can reduce email threats by applying these measures:

- Verify sender addresses and unusual requests.
    
- Avoid opening unexpected attachments.
    
- Inspect links before visiting unfamiliar websites.
    
- Enable MFA on email accounts.
    
- Use secure email gateways and spam filters.
    
- Implement SPF, DKIM, and DMARC for domain-level email authentication and spoofing protection.
    
- Keep email clients, browsers, and operating systems updated.
    
- Confirm payment requests using a separate trusted channel.
    
- Report suspicious emails to the appropriate security team.
    
- Provide regular security awareness training.
    

### 7. Incident Response

If an account or device is compromised, the user or organization should:

1. Disconnect an infected device from the network when appropriate.
    
2. Report the incident to the security team.
    
3. Change compromised passwords from a trusted device and revoke suspicious sessions.
    
4. Preserve relevant logs and evidence.
    
5. Remove the cause of the compromise and patch affected systems.
    
6. Restore affected systems from clean backups when necessary.
    
7. Review the incident and strengthen security controls.


---

# UNIT II — Security in Operating Systems and Networks

Below are exam-ready answers to all questions visible in your second page, organized into short, medium, and long answers.

## A. Short Answer Questions

### 1. What is operating system security?

Operating system security is the protection of an operating system, its files, processes, memory, and resources against unauthorized access, malware, and other attacks.

Example: User permissions, firewalls, and regular security updates help protect an operating system.

### 2. What is a rootkit?

A rootkit is a type of malicious software designed to hide its presence while allowing an attacker to maintain unauthorized access or privileged control over a system.

Example: A kernel-level rootkit may hide malicious processes and files from normal system monitoring tools.

### 3. What is a Denial-of-Service (DoS) attack?

A Denial-of-Service (DoS) attack attempts to make a computer system, website, or network service unavailable by overwhelming it with traffic or exhausting its resources.

Example: Sending excessive requests to a website until it becomes unavailable to legitimate users.

### 4. What is Distributed Denial-of-Service (DDoS)?

A Distributed Denial-of-Service (DDoS) attack uses multiple compromised devices or systems to overwhelm a target with traffic or requests, making its services unavailable.

Example: A botnet sends large volumes of requests to an online shopping website, preventing customers from accessing it.

Difference: A DoS attack may originate from a single source, whereas a DDoS attack uses multiple distributed sources.

## B. Medium Answer Questions

### 1. Explain the importance of security in operating systems.

Operating system security protects the computer's resources and provides a secure environment for applications and users.

Its importance includes:

1. Data protection: Prevents unauthorized access to personal and confidential files.
    
2. Access control: Ensures users can access only permitted resources.
    
3. Malware prevention: Helps protect against viruses, ransomware, spyware, and other malicious programs.
    
4. Process isolation: Prevents one process from improperly accessing another process's memory or resources.
    
5. System integrity: Protects operating system files and configurations from unauthorized modification.
    
6. Network protection: Helps control incoming and outgoing network communications.
    
7. Availability: Reduces the risk of system crashes, disruptions, and resource abuse.
    
8. Privacy: Protects sensitive user information from unauthorized disclosure.
    

Example: Linux file permissions prevent ordinary users from modifying protected system files without the required privileges.

Conclusion: Operating system security is essential for maintaining confidentiality, integrity, availability, and reliable system operation.

### 2. Explain the major security considerations in the design of operating systems.

Operating systems should be designed with security mechanisms that protect resources and enforce access restrictions.

Major considerations:

1. Authentication: Verify user identities through passwords, security keys, or other methods.
    
2. Access control: Restrict access to files, devices, processes, and system resources.
    
3. Least privilege: Give users and processes only the permissions necessary for their tasks.
    
4. Process isolation: Keep processes separated so that one cannot improperly access another's memory.
    
5. Memory protection: Prevent unauthorized access to protected memory regions.
    
6. Secure boot: Verify trusted software components during system startup.
    
7. Encryption: Protect sensitive data stored on disks and other storage devices.
    
8. Auditing and logging: Record security-relevant events for investigation and monitoring.
    
9. Patch management: Correct known vulnerabilities through timely updates.
    
10. Secure defaults: Disable unnecessary services and use restrictive default permissions.
    

Example: A well-configured operating system runs everyday applications with ordinary user privileges instead of administrator or root privileges.

Conclusion: Security must be incorporated into the operating system's architecture and maintained through secure configuration, updates, and monitoring.

### 3. What is a rootkit? Explain its general purpose and impact on system security.

A rootkit is malicious software or a collection of components designed to conceal unauthorized activity and maintain access to a compromised system.

General purposes of rootkits:

- Hide malicious files and processes.
    
- Conceal network connections.
    
- Maintain persistent unauthorized access.
    
- Evade antivirus and security monitoring.
    
- Help attackers retain privileged control.
    

Types of rootkits:

1. User-mode rootkit: Operates in user space and may manipulate applications or system utilities.
    
2. Kernel-mode rootkit: Operates with kernel-level privileges and can interfere with core operating system functions.
    
3. Bootkit: Targets the boot process to load malicious components before or during operating system startup.
    
4. Firmware rootkit: Targets device firmware to establish persistent compromise.
    

Impact on system security:

- Theft of sensitive information.
    
- Concealment of malware and attacker activity.
    
- Unauthorized modification of system behavior.
    
- Disabling or bypassing security controls.
    
- Long-term compromise that is difficult to detect.
    

Detection and prevention:

- Keep the operating system updated.
    
- Use trusted endpoint security software.
    
- Enable Secure Boot where supported.
    
- Monitor system behavior and security logs.
    
- Verify system integrity using trusted tools.
    
- Reinstall the operating system from trusted media if a serious compromise cannot be reliably removed.
    

Conclusion: Rootkits are dangerous because they conceal malicious activity and can undermine the tools used to detect and remove threats.

### 4. Explain network security attacks, threats to network communications and wireless network security.

Network security protects data, devices, and services communicating across networks against unauthorized access, interception, modification, and disruption.

A. Common network attacks

1. Eavesdropping: Secretly monitoring network communications to obtain information.
    
2. Man-in-the-Middle (MitM): Intercepting or potentially modifying communication between two parties.
    
3. IP spoofing: Forging a source IP address to disguise the apparent origin of traffic.
    
4. DNS spoofing: Providing false DNS information to redirect users to an unintended destination.
    
5. Packet sniffing: Capturing network packets for analysis; it becomes a security threat when used to access data without authorization.
    
6. DoS and DDoS: Overwhelming network services or exhausting resources to cause disruption.
    
7. Password attacks: Attempting to guess or steal credentials used to access network resources.
    

B. Threats to network communications

- Unencrypted data may be exposed during transmission.
    
- Weak authentication may allow unauthorized connections.
    
- Misconfigured network devices may expose internal resources.
    
- Malware can spread between connected systems.
    
- Rogue devices can intercept traffic or introduce malicious activity.
    
- Vulnerable protocols and outdated firmware can be exploited.
    

C. Wireless network security

Wireless networks use radio signals, making it important to protect both access to the network and transmitted data.

Common wireless threats include:

- Rogue access points: Unauthorized wireless access points connected to a network.
    
- Evil twin attacks: Fake Wi-Fi access points impersonating legitimate networks.
    
- Wi-Fi password attacks: Attempts to guess or recover wireless credentials.
    
- Wireless eavesdropping: Monitoring wireless traffic.
    
- Deauthentication attacks: Disrupting wireless connections by abusing management frames or protocol weaknesses.
    

Protection measures:

- Use WPA3 where supported, or WPA2-AES where WPA3 is unavailable.
    
- Set a strong, unique Wi-Fi password.
    
- Change default router administrator credentials.
    
- Update router firmware regularly.
    
- Disable WPS if it is unnecessary or insecurely configured.
    
- Use a guest network for visitors and untrusted devices.
    
- Avoid sensitive activity on untrusted public Wi-Fi, or use a trusted VPN when appropriate.
    
- Monitor connected devices and remove unauthorized access points.
    

Conclusion: Network security requires secure communication protocols, proper access control, updated network devices, traffic monitoring, and strong wireless security configurations.

## C. Long Answer Questions

### 1. Explain operating system security in detail. Discuss security in OS design, common security threats and rootkits.

### Introduction

An operating system (OS) manages computer hardware, memory, processes, files, applications, and devices. Operating system security protects these resources from unauthorized access, malicious software, data theft, and disruption.

Examples of operating systems include Linux, Windows, and macOS.

### 1. Objectives of Operating System Security

Operating system security supports the following objectives:

- Confidentiality: Prevent unauthorized disclosure of information.
    
- Integrity: Prevent unauthorized modification of files, programs, and system settings.
    
- Availability: Keep systems and resources available to authorized users.
    
- Authentication: Verify user and system identities.
    
- Authorization: Determine which operations an authenticated user can perform.
    
- Accountability: Record security-relevant activity so actions can be investigated.
    

### 2. Security Mechanisms in an Operating System

a. User accounts and authentication

Operating systems provide user accounts and authentication mechanisms to control access. Administrator or root accounts have elevated privileges and require special protection.

b. File permissions

File permissions specify who can read, write, or execute files.

For example, Linux distinguishes among the file owner, group, and other users. Appropriate permissions prevent ordinary users from modifying protected files.

c. Process isolation

Each process should be restricted to its permitted memory and resources. Process isolation limits the damage caused by application errors and compromises.

d. Memory protection

Memory protection prevents unauthorized processes from reading or modifying protected memory regions.

e. Firewall and network controls

Host firewalls restrict network connections according to defined rules. Disabling unnecessary network services also reduces the attack surface.

f. Updates and patching

Security patches fix known vulnerabilities in the operating system and its components.

g. Encryption

Full-disk encryption helps protect stored data if a device is lost or stolen. Encryption keys must also be protected.

h. Logging and auditing

System logs record events such as authentication attempts, privilege changes, and security alerts. These records support detection and incident investigation.

### 3. Security Considerations in OS Design

A secure operating system should follow several design principles.

1. Least privilege: Give users and processes only the access required for their functions.
    
2. Defense in depth: Use multiple layers of security instead of relying on a single control.
    
3. Secure defaults: Begin with restrictive settings and disable unnecessary services.
    
4. Complete mediation: Check authorization whenever access to a protected resource is requested, as appropriate to the system design.
    
5. Separation of privilege: Avoid granting excessive authority based on a single condition or credential.
    
6. Secure boot: Verify trusted components during startup to help prevent boot-level tampering.
    
7. Isolation: Separate user processes, system services, and sensitive components.
    
8. Auditability: Maintain useful logs and integrity checks.
    
9. Secure updates: Verify updates and deliver patches through trusted mechanisms.
    
10. Recovery: Provide reliable backups and recovery procedures for serious incidents.
    

### 4. Common Operating System Security Threats

a. Malware

Malware includes viruses, worms, Trojans, spyware, ransomware, and rootkits. It can steal information, damage files, or provide attackers with unauthorized access.

b. Privilege escalation

Privilege escalation occurs when an attacker gains permissions beyond those originally authorized.

- Vertical escalation: A normal user obtains administrator or root privileges.
    
- Horizontal escalation: A user gains access to another user's resources at a similar privilege level.
    

c. Password attacks

Attackers may use phishing, credential theft, brute force, or reused passwords to gain access to accounts.

d. Unpatched vulnerabilities

Outdated operating systems and software may contain known weaknesses that attackers can exploit.

e. Misconfiguration

Incorrect permissions, exposed services, weak security policies, and default credentials can make systems vulnerable.

f. Denial-of-Service attacks

An attacker may consume CPU, memory, storage, or network capacity to prevent legitimate use of the system.

g. Insider threats

Authorized users may intentionally or accidentally expose sensitive data, misuse privileges, or weaken system security.

### 5. Rootkits

A rootkit is a type of malware designed to hide malicious activity and maintain unauthorized access to a system.

Types of rootkits

- User-mode rootkits: Manipulate user-space programs or libraries.
    
- Kernel-mode rootkits: Interfere with operating system kernel functions.
    
- Bootkits: Target boot components to load malicious code early in startup.
    
- Firmware rootkits: Compromise firmware in supported hardware components.
    

How rootkits affect a system

Rootkits may hide processes, files, registry entries, or network connections. Some can interfere with security software and system monitoring tools.

A kernel-level rootkit is particularly dangerous because it operates with high privileges and may manipulate the operating system's view of the system.

Rootkit prevention and detection

- Install operating system and firmware updates.
    
- Use reputable endpoint security tools.
    
- Enable Secure Boot and other platform security features where supported.
    
- Restrict administrator access.
    
- Monitor unusual processes, network connections, and system changes.
    
- Verify system integrity using trusted offline tools or a trusted recovery environment.
    

If a rootkit has deeply compromised the operating system, ordinary malware removal may not be sufficient. A clean reinstallation from trusted media, and in some cases firmware recovery, may be required.

### 6. Best Practices for Operating System Security

Organizations and individual users should:

1. Use strong authentication and MFA where supported.
    
2. Apply security updates promptly.
    
3. Use least privilege and separate administrator accounts.
    
4. Enable host firewalls and disable unnecessary services.
    
5. Encrypt sensitive stored information.
    
6. Maintain regular, tested backups.
    
7. Review security logs and alerts.
    
8. Install software from trusted sources.
    
9. Use secure boot features where available.
    
10. Maintain an incident response and recovery plan.
    

### Conclusion

Operating system security is fundamental to protecting computer resources and information. Secure design, authentication, access control, process isolation, patching, encryption, and monitoring reduce the risk of compromise. Rootkits and other advanced threats require particular attention because they may conceal malicious activity and undermine normal security mechanisms.

### 2. Explain network security attacks in detail. Discuss threats to network communications, wireless network security, DoS and DDoS attacks.

### Introduction

Network security protects network infrastructure, connected devices, communications, and transmitted data from unauthorized access, interception, modification, and disruption.

It uses security controls such as firewalls, encryption, authentication, intrusion detection, network segmentation, and traffic monitoring.

### 1. Threats to Network Communications

a. Eavesdropping

Eavesdropping involves monitoring communications to obtain sensitive information such as credentials or confidential messages.

Protection: Use properly configured encryption protocols such as TLS and secure wireless encryption.

b. Man-in-the-Middle (MitM) attack

An attacker positions themselves between two communicating parties and may intercept or alter traffic.

Protection: Validate TLS certificates, use trusted networks, and avoid ignoring certificate warnings.

c. IP spoofing

IP spoofing involves forging the source IP address in network packets. It can be used to disguise traffic origins or support certain denial-of-service and impersonation attacks.

Protection: Use ingress and egress filtering, anti-spoofing rules, and suitable network architecture.

d. DNS attacks

DNS attacks may involve forged responses, cache poisoning, or unauthorized modification of DNS records to redirect users.

Protection: Secure DNS administration, use DNSSEC where appropriate, and monitor DNS records and traffic.

e. Packet sniffing

Packet sniffing captures and analyzes network traffic. It is a legitimate troubleshooting technique but can be abused to inspect unauthorized communications.

Protection: Encrypt sensitive traffic and restrict access to network monitoring tools.

f. Replay attacks

A replay attack reuses previously captured authentication messages or transactions to attempt an unauthorized action.

Protection: Use nonces, timestamps, sequence numbers, and secure protocols that prevent the reuse of old messages.

g. Malware and unauthorized access

Compromised network devices or endpoints can be used to steal data, spread malware, or access internal resources.

Protection: Apply endpoint security, patch systems, enforce access control, and segment networks.

### 2. Wireless Network Security

Wireless networks transmit data through radio signals, so access control and encryption are particularly important.

Common wireless threats

a. Evil twin: A fake access point imitates a legitimate Wi-Fi network to trick users into connecting.

b. Rogue access point: An unauthorized access point creates an uncontrolled path into a network.

c. Password attacks: Weak wireless passwords may be guessed or recovered through attacks against authentication mechanisms.

d. Wireless eavesdropping: An attacker monitors wireless traffic, potentially exposing information that is not properly encrypted.

e. Deauthentication attacks: An attacker abuses wireless management-frame mechanisms to disrupt client connections.

f. Outdated encryption: Legacy security protocols and insecure configurations may expose wireless communications to attack.

Wireless security measures

- Use WPA3 or WPA2-AES with a strong password.
    
- Change default router credentials.
    
- Keep router firmware updated.
    
- Disable unnecessary WPS functionality.
    
- Use guest networks to isolate visitors.
    
- Use enterprise authentication, such as WPA2-Enterprise or WPA3-Enterprise, when appropriate.
    
- Enable protected management frames where supported.
    
- Monitor connected devices and investigate unauthorized access points.
    
- Avoid automatically connecting to unfamiliar public Wi-Fi networks.
    

### 3. Denial-of-Service (DoS) Attacks

A Denial-of-Service attack attempts to make a service unavailable to legitimate users by exhausting resources or exploiting weaknesses.

Common categories include:

a. Volumetric attacks: Consume available bandwidth with excessive traffic.

b. Protocol attacks: Exhaust resources in network infrastructure or protocol implementations.

c. Application-layer attacks: Overload applications with large numbers of requests or expensive operations.

Effects of DoS attacks

- Website or service downtime.
    
- Failed transactions.
    
- Loss of productivity.
    
- Financial losses.
    
- Reduced customer trust.
    

Protection measures

- Configure firewalls and rate limits.
    
- Monitor network traffic and system resources.
    
- Use load balancing and resilient infrastructure.
    
- Apply appropriate filtering and connection limits.
    
- Maintain an incident response plan.
    

### 4. Distributed Denial-of-Service (DDoS) Attacks

A DDoS attack uses many distributed systems to send traffic or requests toward a target. The attacking systems may include compromised computers, servers, routers, or Internet of Things devices forming a botnet.

How a DDoS attack works

1. An attacker gains control of or coordinates multiple devices.
    
2. Those devices send traffic or requests toward the target.
    
3. The combined traffic overwhelms bandwidth, network equipment, or application resources.
    
4. Legitimate users experience slow performance or service unavailability.
    

Types of DDoS attacks

- Volumetric attacks: Attempt to saturate the target's available bandwidth.
    
- Protocol attacks: Exhaust resources on network infrastructure or protocol handlers.
    
- Application-layer attacks: Send large numbers of requests designed to consume application or server resources.
    

Difference between DoS and DDoS

|Feature|DoS|DDoS|
|---|---|---|
|Source|Typically one source|Multiple distributed sources|
|Traffic volume|May be limited|Can be very large|
|Mitigation|May be simpler to isolate|Often requires distributed filtering|
|Attribution|Source may be easier to identify|More difficult because many systems are involved|

DDoS protection measures

1. Use a reputable DDoS protection or traffic-scrubbing service.
    
2. Apply rate limiting and filtering.
    
3. Use content delivery networks (CDNs) and load balancing where appropriate.
    
4. Monitor traffic patterns and establish alerts.
    
5. Coordinate with the hosting provider or ISP during an incident.
    
6. Protect application endpoints and limit expensive operations.
    
7. Maintain a documented response plan and test it periodically.
    

No single defense can prevent every DDoS attack. Effective mitigation depends on the attack type, the target's architecture, and the capacity of upstream network providers.

### 5. Role of Firewalls and Intrusion Detection Systems

Firewall: Controls network traffic according to configured security rules. It can restrict connections between trusted and untrusted networks.

Intrusion Detection System (IDS): Monitors network or host activity for suspicious behavior and generates alerts.

Intrusion Prevention System (IPS): Can detect and actively block certain malicious traffic.

These controls complement encryption, access control, network segmentation, and continuous monitoring.

### 6. General Network Security Best Practices

- Use secure protocols and encryption.
    
- Keep network equipment and endpoints updated.
    
- Use strong authentication and MFA.
    
- Segment sensitive systems from less-trusted networks.
    
- Restrict unnecessary ports and services.
    
- Monitor logs, traffic, and security alerts.
    
- Maintain secure backups.
    
- Conduct authorized security assessments.
    
- Prepare an incident response plan.
    

### Conclusion

Network security protects communications and services from interception, unauthorized access, and disruption. Wireless encryption, secure protocols, firewalls, monitoring, and proper access control help protect ordinary network traffic. DoS and DDoS defenses additionally require traffic filtering, rate limiting, resilient infrastructure, and coordination with service providers.

These answers cover all 14 questions shown on page 2: four short-answer, four medium-answer, and two long-answer questions, with the long-answer sections addressing the combined topics specified in the assignment.