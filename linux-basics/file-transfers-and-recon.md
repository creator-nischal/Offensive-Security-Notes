# File Transfers & Web Recon Notes

Quick reference for moving files between target machines and my Kali system during labs/assessments, plus web directory enumeration.

---

## 1. Moving Files Between Target Server & Kali

### Method 1: Python HTTP Server + `wget` / `curl`

**On Target Server / Host (serving the file):**
```bash
# Navigate to the folder containing the file
python3 -m http.server 8000
```

**On Kali Machine (downloading the file):**
```bash
# Using wget
wget http://<TARGET_IP>:8000/filename

# OR using curl (-o saves output to file)
curl http://<TARGET_IP>:8000/filename -o filename
```

---

### Method 2: Secure Copy (SCP)
When SSH access or credentials are available:
```bash
scp filename user@<TARGET_IP>:/path/to/destination/
```

---

### Method 3: Base64 Encoding (No Network Listener Needed)
Great for transferring small binary/script files over raw shells when HTTP or SCP is blocked.

**On Source Server:**
```bash
# Encode file to single line base64 string
base64 shell -w 0
```

**On Target Machine:**
```bash
# Decode base64 string back into binary file
echo "PASTE_BASE64_STRING_HERE" | base64 -d > shell
chmod +x shell
```

---

## 2. Web Footprinting & Directory Bruteforcing

### Directory Bruteforcing with Gobuster
Finding hidden admin panels, backup folders, or uploaded content on a target web server:

```bash
gobuster dir -u http://<TARGET_IP>/path -w /usr/share/seclists/Discovery/Web-Content/common.txt
```
