# task 1
- get the ip of the machine by this cmd :- `arp-scan -l`
![[Pasted image 20260303144139.png]]
- i got the ip of machine ip :- 192.168.248.131
# task 2
now we have ip addr so we starts enumarating on it using nmap.
cmd :- `nmap -T4 -sV -sC -O 192.168.248.131 -vv -o nmap`
and here is output of nmap :- 
```nmap
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:e3:f6:dc:2e:22:5d:17:ac:46:02:39:ad:71:cb:49 (DSA)
|_  2048 9a:82:e6:96:e4:7e:d6:a6:d7:45:44:cb:19:aa:ec:dd (RSA)
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
|_http-server-header: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
|_http-title: Ligoat Security - Got Goat? Security ...
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
MAC Address: 00:0C:29:A8:EB:39 (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
- we have open ports 22,80 and some os information and kernal info.
- we have open port so i starts with port 80 beacuse of i don't have any user passwd for ssh.
- 