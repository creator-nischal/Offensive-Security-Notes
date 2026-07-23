# Dev Notes & Offensive Security Vault ⚡

Welcome to my personal learning vault! This repository is my living technical notebook where I record everything I learn, test, and break in security, Linux, penetration testing, and web exploitation.

No corporate fluff, no AI-generated filler—just practical, real-world notes, lab writeups, command references, and privilege escalation techniques I've actually tested.

---

## 🗂️ Table of Contents

### 1. 🐧 Linux & Recon Fundamentals
- [File Transfers & Web Footprinting](linux-basics/file-transfers-and-recon.md)
  - Python HTTP server, `wget`, `curl`, `scp`, `base64` transfers.
  - Web directory enumeration using `gobuster`.

### 2. ⚡ Linux Privilege Escalation
- [Linux PrivEsc Methodology & Enumeration Handbook](privilege-escalation/linux-privesc-handbook.md)
  - Situational awareness (`whoami`, `id`, `PATH`, kernel).
  - Checking sudo permissions (`sudo -l`), Cron jobs, SUID/SGID binaries.
  - Searching for readable secrets (`id_rsa`, `/etc/shadow`, credentials in `.conf` files).
- [Real-World PrivEsc Case Studies & Exploits](privilege-escalation/real-world-escalations.md)
  - Case 1: Switching users via NOPASSWD bash (`sudo -u user2 /bin/bash`).
  - Case 2: Root SSH key extraction (`id_rsa`).
  - Case 3: Hijacking writable root scripts (`monitor.sh`).
  - Case 4: Sudo PHP inline shell execution (`sudo php -r ...`).

### 3. 🎯 Web Exploitation & Reverse Shells
- [Reverse Shells & File Upload Filter Bypasses](web-security/reverse-shells-and-uploads.md)
  - PHP reverse shell payloads (`/dev/tcp` execution).
  - Netcat listeners (`nc -lvnp`).
  - Extension filter bypasses (`.php` -> `.phar`).

### 4. 🧪 Lab Writeups & Machine Notes
- [GetSimple HTB Lab Walkthrough](htb-labs/getsimple-writeup.md)
  - Footprinting, XML user data extraction, password hash cracking, admin access, file upload RCE.

---

## 💡 Core Philosophy
- **Learn by Doing**: Every command here was run in real labs (HTB, THM, local boxes).
- **Understand the "Why"**: Don't just copy-paste payloads—understand how system calls, interpreters, and misconfigurations work under the hood.
- **Continuous Growth**: Constantly updated as I learn new techniques.
