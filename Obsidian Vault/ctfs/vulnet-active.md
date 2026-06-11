- started with nmap:
- scan no 1
```nmap
sudo nmap -Pn $ip -o vulnet-active/namp.txt         
[sudo] password for user: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-09 10:51 +0530
Stats: 0:00:04 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 38.15% done; ETC: 10:51 (0:00:06 remaining)
Nmap scan report for 10.49.185.207 (10.49.185.207)
Host is up (0.017s latency).
Not shown: 995 filtered tcp ports (no-response)
PORT    STATE SERVICE
53/tcp  open  domain
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
464/tcp open  kpasswd5

Nmap done: 1 IP address (1 host up) scanned in 6.17 seconds

```

- scan no 2
```nmap
sudo nmap -sT -T4 -sV -sC -Pn $ip -p53,135,139,445,464 -o nmap2.txt
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-09 10:54 +0530
Nmap scan report for 10.49.185.207 (10.49.185.207)
Host is up (0.013s latency).

PORT    STATE SERVICE       VERSION
53/tcp  open  domain        Simple DNS Plus
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
464/tcp open  kpasswd5?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-06-09T05:24:45
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.87 seconds
```

- scan no 3
```nmap
sudo nmap -Pn $ip -p- -o nmap3.txt
Nmap scan report for 10.49.185.207 (10.49.185.207)
Host is up, received user-set (0.014s latency).
Scanned at 2026-06-09 10:57:31 IST for 108s
Not shown: 65520 filtered tcp ports (no-response)
PORT      STATE SERVICE      REASON
53/tcp    open  domain       syn-ack ttl 126
135/tcp   open  msrpc        syn-ack ttl 126
139/tcp   open  netbios-ssn  syn-ack ttl 126
445/tcp   open  microsoft-ds syn-ack ttl 126
464/tcp   open  kpasswd5     syn-ack ttl 126
6379/tcp  open  redis        syn-ack ttl 126
9389/tcp  open  adws         syn-ack ttl 126
49666/tcp open  unknown      syn-ack ttl 126
49667/tcp open  unknown      syn-ack ttl 126
49669/tcp open  unknown      syn-ack ttl 126
49670/tcp open  unknown      syn-ack ttl 126
49671/tcp open  unknown      syn-ack ttl 126
49678/tcp open  unknown      syn-ack ttl 126
49691/tcp open  unknown      syn-ack ttl 126
49785/tcp open  unknown      syn-ack ttl 126

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 107.96 seconds
           Raw packets sent: 131134 (5.770MB) | Rcvd: 93 (4.092KB)
```

- scan no 4
```nmap
 sudo nmap -sT -T4 -sV -sC -Pn $ip -p6379,9389,49666,49667,49669,49670,49671,49678,49691,49685 -o nmap4.txt
[sudo] password for user: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-09 11:04 +0530
Nmap scan report for 10.49.185.207 (10.49.185.207)
Host is up (0.019s latency).

PORT      STATE    SERVICE    VERSION
6379/tcp  open     redis      Redis key-value store 2.8.2402
9389/tcp  open     mc-nmf     .NET Message Framing
49666/tcp open     msrpc      Microsoft Windows RPC
49667/tcp open     msrpc      Microsoft Windows RPC
49669/tcp open     msrpc      Microsoft Windows RPC
49670/tcp open     ncacn_http Microsoft Windows RPC over HTTP 1.0
49671/tcp open     msrpc      Microsoft Windows RPC
49678/tcp open     msrpc      Microsoft Windows RPC
49685/tcp filtered unknown
49691/tcp open     msrpc      Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 60.75 seconds

```

---
 - enuming smb
 - but nothing got in my hands
 ```
 Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-06-09T05:24:45
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.87 seconds

 ```

---

- now enuming redis 
  - by metasploit:
```
msfconsole         
Metasploit tip: Writing a custom module? After editing your module, why not try 
the reload command
                                                  

         .                                         .
 .

      dBBBBBBb  dBBBP dBBBBBBP dBBBBBb  .                       o
       '   dB'                     BBP
    dB'dB'dB' dBBP     dBP     dBP BB
   dB'dB'dB' dBP      dBP     dBP  BB
  dB'dB'dB' dBBBBP   dBP     dBBBBBBB

                                   dBBBBBP  dBBBBBb  dBP    dBBBBP dBP dBBBBBBP
          .                  .                  dB' dBP    dB'.BP
                             |       dBP    dBBBB' dBP    dB'.BP dBP    dBP
                           --o--    dBP    dBP    dBP    dB'.BP dBP    dBP
                             |     dBBBBP dBP    dBBBBP dBBBBP dBP    dBP

                                                                    .
                .
        o                  To boldly go where no
                            shell has gone before


       =[ metasploit v6.4.134-dev                               ]
+ -- --=[ 2,650 exploits - 1,336 auxiliary - 2,141 payloads     ]
+ -- --=[ 432 post - 49 encoders - 14 nops - 12 evasion         ]

Metasploit Documentation: https://docs.metasploit.com/
The Metasploit Framework is a Rapid7 Open Source Project

msf > use auxiliary/scanner/redis/redis_server
msf auxiliary(scanner/redis/redis_server) >  set rhosts 10.49.185.207
rhosts => 10.49.185.207
msf auxiliary(scanner/redis/redis_server) > exploit
[*] 10.49.185.207:6379    - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf auxiliary(scanner/redis/redis_server) > 


```

 - now with redis-cli