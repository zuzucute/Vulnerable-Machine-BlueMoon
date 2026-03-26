# Vulnerable-Machine-BlueMoon

# BlueMoon 2021 VulnHub Walkthrough

## 🧠 Objective

* Identify target machine
* Enumerate open services
* Gain initial access
* Escalate privileges to root

---

## 🖥️ Lab Setup

* Attacker: Kali Linux
* Attacker IP : 192.168.56.101
* Target: BlueMoon 2021 (VulnHub)
* Target IP : 192.168.56.102

---

## 🔍 Step 1 — Discover Attacker IP

I used ifconfig to know the attacker's IP.

ifconfig

## 🔍 Step 2 — Discover Target IP

And then, identify the target machine on the network.

sudo netdiscover  192.168.56.0/24

📌 Look for a new IP that is not your Kali machine.

Example result:

192.168.56.102 → Target Machine

---

## 🚪 Step 3 — Port Scanning

Run a full scan to identify open ports and services.

nmap -sC -sV -Pn -vv 192.168.56.102


### Example Findings:

* Port 22 → SSH
* Port 21 → FTP
* Port 80 → HTTP (Web Server)

## 🌐 Step 4 — Web Enumeration

I used gobuster to perform brute-forcing command to discover hidden paths on a web server.

gobuster dir --url http://ip -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 25 -x .txt, .php, .bak, .zip, .html

These are the directories that has been succesfully identified.

* /service-status
* /hiddent_text

- Accessing the root URL at http://192.168.56.102/hidden_text that led to a maintenance notice which stating, "Maintenance!Sorry For Delay.We will recover soon. Thank You ...
- There is something off with the Thank You and after clicking it, I discovered a QR code.
- Decoding the QR code revealed a FTP credentials for user and password.

user : user ftp passsword : fttp@ssword 


## 🔑 Step 5 — FTP Access

I
## 🐚 Step 5 — Stabilize Shell

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

---

## 🔎 Step 6 — Enumeration (Post-Exploitation)

Check:

```bash
whoami
id
uname -a
```

Look for:

* SUID files
* Writable files
* Credentials

---

### SUID Check

```bash
find / -perm -4000 2>/dev/null
```

---

### Check for Credentials

```bash
cat /etc/passwd
cat /etc/shadow
```

Look in:

```
/var/www/
/home/
```

---

## 🔼 Step 7 — Privilege Escalation

### Check sudo permissions:

```bash
sudo -l
```

If something runs as root without password → exploit it.

---

### Example Exploit (SUID Binary)

If a vulnerable binary exists:

```bash
./binary
```

Or use GTFOBins:
[https://gtfobins.github.io/](https://gtfobins.github.io/)

---

### Cron Jobs

```bash
cat /etc/crontab
```

If writable script is executed → inject reverse shell.

---

## 👑 Step 8 — Root Access

Once escalated:

```bash
whoami
```

Expected:

```
root
```

---

## 🏁 Flags

Search for flags:

```bash
find / -name "*flag*" 2>/dev/null
```

Common locations:

* `/root`
* `/home/user`

---

## 📌 Key Takeaways

* Always enumerate thoroughly before exploiting
* Web apps are common entry points
* Misconfigured permissions → easy privilege escalation
* SUID + sudo = privilege escalation goldmine

---

## ⚠️ Notes

* This machine focuses on:

  * Web exploitation
  * Basic enumeration
  * Privilege escalation fundamentals

---

## ✅ Tools Used

* `netdiscover`
* `nmap`
* `gobuster`
* `nc`
* `python3`
* `GTFOBins`

---

## 🎯 Conclusion

BlueMoon 2021 is a beginner-friendly machine that teaches:

* Network discovery
* Service enumeration
* Web exploitation basics
* Privilege escalation techniques

---

Happy hacking 🚀

If you want, I can customize this with the **exact commands/output from your machine** so your GitHub write-up looks more real and professional.


