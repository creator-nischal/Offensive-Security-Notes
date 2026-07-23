# Linux Post-Exploitation & PrivEsc Enumeration

The complete checklist to run when landing on a target Linux box.

---

## 1. Fast Situational Awareness
```bash
whoami     # Current user
id         # Group memberships
hostname   # Target host name
ip a       # Network interfaces & subnets
sudo -l    # Check passwordless sudo commands (NOPASSWD)
```

---

## 2. System & Kernel Profiling
```bash
cat /etc/os-release   # OS distribution and version
uname -a              # Kernel release (search for public kernel exploits)
echo $PATH            # Check for binary execution paths
env                   # View user environment variables
cat /etc/shells       # Check valid system login shells
```

---

## 3. High-Value Targets
- **SSH Keys**: Search `/home/*/.ssh/id_rsa` for readable private keys.
- **Process List**: `ps aux | grep root` (look for root services with known vulnerabilities).
- **Cron Jobs**: `cat /etc/crontab` and `ls -la /etc/cron.daily/`.
- **Writable Paths**:
  - `find / -path /proc -prune -o -type d -perm -o+2 2>/dev/null` (Writable directories)
  - `find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null` (Writable files)
