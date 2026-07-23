# File Upload Bypasses

Methods for bypassing web application file upload filters to drop shell payloads on a server.

---

## Extension Bypasses

### `.php` Filter Bypass -> `.phar`
When a web app blocks standard `.php` files:
- Rename the extension from `.php` to `.phar` (PHP Archive).
- Modern PHP interpreters execute `.phar` files as standard PHP scripts.

**Example (GetSimple CMS)**:
1. Renamed `shell.php` to `shell.phar`.
2. Uploaded successfully through the user asset panel.
3. Triggered shell execution by visiting `/data/uploads/shell.phar`.

---

## Plugin / Admin Upload Exploitation

### Nibbleblog Exploitation
1. Logged into admin dashboard.
2. Located plugin upload feature (`my_image` plugin).
3. Uploaded custom PHP payload directly.
4. Navigated to `/content/private/plugins/my_image/image.php` to execute.
