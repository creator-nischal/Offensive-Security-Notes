# Linux Privilege Escalation Methodology & Enumeration Handbook

My core checklist and mindset whenever I land initial access on a Linux target.

---

## 🧠 Mindset & Golden Rule
> **Rule of Thumb**: Don't panic if the server OS or shell isn't what you're used to. Keep calm and run standard enumeration checks. Your goal is to find **one misconfiguration or vulnerability** (via `sudo -l`, SUIDs, writable scripts, internal services, etc.) and leverage it to escalate.

---

## 1. Initial Situational Awareness (First 60 Seconds)

Right after dropping into a shell, run these commands to know where you are:

```bash
whoami     # What user am I running as?
id         # What groups does this user belong to?
hostname   # Target hostname (can reveal server function)
ip a       # Check subnets & additional NICs (dual-homed hosts)
sudo -l    # Check sudo rights! (Can we drop directly into root with 'sudo su'?)
```

---

## 2. System Identification & Environment

```bash
# OS details & version (check for known OS exploits)
cat /etc/os-release

# Kernel version (check searchsploit / exploit-db for kernel exploits)
uname -a

# Check system PATH environment variable
echo $PATH

# View full environment variables for secrets
env

# CPU architecture details
lscpu

# Available login shells
cat /etc/shells
```

---

## 3. Enumerating Users, History & Credentials

### Running Processes
```bash
# Processes running as root
ps aux | grep root

# Current terminal-attached processes
ps au
```

### User Home Directories & History
Check home folders for command history or leftover secrets:
```bash
# View command history
history
cat ~/.bash_history

# Check for SSH private keys (.ssh folder)
# If id_rsa exists and is readable, we can copy it and SSH directly into the user/root!
ls -la /home/*/.ssh/
```

### Network Connections & ARP Cache
```bash
# Check ARP table to see other hosts communicating with this box
arp -a
```

---

## 4. Searching Secrets in Files & System Configs

- **Config Files**: Search `.conf` and `.config` files for cleartext passwords and database credentials.
- **Readable `/etc/shadow`**: If shadow is readable by low-priv users, extract password hashes for offline cracking (`john` or `hashcat`).
- **Hashes in `/etc/passwd`**: Sometimes legacy systems store password hashes directly in `/etc/passwd` instead of `x`.

---

## 5. Cron Jobs & Automation Tasks

Cron jobs run automatically in the background (often as root). Look for misconfigured relative paths or weak script permissions:

```bash
ls -la /etc/cron.daily/
cat /etc/crontab
```

---

## 6. Storage & Writable Paths

### Drives & Mounts
```bash
# Unmounted drives or extra file systems that might contain backups
lsblk
```

### Finding Writable Directories & Files
Great for transferring enumeration scripts (`linpeas.sh`) or hijacking cron scripts:

```bash
# Find writable directories
find / -path /proc -prune -o -type d -perm -o+2 2>/dev/null

# Find writable files
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```
