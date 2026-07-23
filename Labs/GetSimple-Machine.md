# GetSimple Target Machine Notes

- **Target IP**: `10.129.114.111`
- **Open Ports**: 22 (SSH), 80 (HTTP)

---

## Exploitation Workflow
1. **Enumeration**: Nmap revealed OpenSSH 8.2p1 & Apache 2.4.41. `robots.txt` disallows `/admin/`.
2. **Data Leak**: Gobuster uncovered `/data/users/admin.xml` containing SHA-1 password hash `d033e22ae348aeb5660fc2140aec35850c4da997`.
3. **Hash Cracking**: Cracked hash online to `admin`. Logged into `/admin/` panel.
4. **File Upload RCE**: `.php` upload blocked. Bypassed filter by uploading `.phar` extension payload to execute remote shell.
