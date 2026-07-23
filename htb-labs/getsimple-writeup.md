# HTB Machine Writeup: GetSimple

- **Target IP**: `10.129.114.111`
- **OS**: Linux (Ubuntu Apache)

---

## 1. Nmap Scan & Enumeration

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/admin/
```

- Port 22 (SSH) and Port 80 (HTTP) open.
- `robots.txt` points to `/admin/`.

---

## 2. Directory Fuzzing & Information Leakage

Running `gobuster` directory scan revealed several sensitive endpoints:

- `/admin/` (Admin Login Panel)
- `/data/` (Directory Indexing enabled)
- `/backups/`
- `/plugins/`

Browsing into `/data/users/admin.xml` leaked administrative account details:

```xml
<item>
  <USR>admin</USR>
  <PWD>d033e22ae348aeb5660fc2140aec35850c4da997</PWD>
  <EMAIL>admin@gettingstarted.com</EMAIL>
</item>
```

---

## 3. Password Hash Cracking & Admin Login

The password hash `d033e22ae348aeb5660fc2140aec35850c4da997` appeared to be SHA-1. 
Cracked the hash online (hashes.com):
- **Decrypted Password**: `admin`
- **Credentials**: `admin` : `admin`

Successfully logged into the GetSimple CMS admin dashboard at `/admin/`.

---

## 4. Exploitation & Initial Shell Access

1. Identified file upload functionality inside theme/plugin management.
2. Direct `.php` upload was restricted, so bypassed extension restrictions by renaming payload to `.phar`.
3. Uploaded `.phar` PHP reverse shell payload and triggered it via web browser.
4. Caught interactive shell on local Netcat listener.
