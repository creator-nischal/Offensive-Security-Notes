# Linux File Transfer Methods

Transferring files between Kali host and target machines.

---

## HTTP Transfers (Python + Wget / Curl)

**Target / Host Serving File:**
```bash
python3 -m http.server 8000
```

**Kali / Receiver:**
```bash
wget http://<TARGET_IP>:8000/filename
# OR
curl http://<TARGET_IP>:8000/filename -o filename
```

---

## SCP Transfer
```bash
scp filename user@<TARGET_IP>:/destination/path/
```

---

## Base64 Transfer (No HTTP needed)
```bash
# On sender machine:
base64 filename -w 0

# On receiver machine:
echo "BASE64_STRING" | base64 -d > filename
```
