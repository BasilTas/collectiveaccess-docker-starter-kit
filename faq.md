# Frequently Asked Questions (FAQ)

This document answers common questions about using the CollectiveAccess Docker Starter Kit. It covers installation, configuration, routing, media handling, upgrades, and general usage patterns.

---

## General Questions

### **What is this starter kit for?**
It provides a modern, reproducible Docker environment for running:

- Providence (backend)
- Pawtucket (frontend)
- MySQL 8.x
- PHP 8.4
- Apache with rewrite enabled

It includes fixes for routing, media handling, and configuration issues that commonly affect CA installations.

---

### **Why Docker instead of manual installation?**
Docker provides:

- consistent environments  
- easy upgrades  
- simple backups  
- no need to install PHP/MySQL/Apache on the host  
- reproducible deployments  
- easy migration between machines  

It also avoids OS‑specific issues (Windows/IIS, macOS permissions, Linux package differences).

---

## Installation Questions

### **Where do I run the installer?**
Providence installer:

http://localhost:8080/ca/install

Code

Pawtucket does not have an installer.

---

### **What database hostname should I use?**
Always use:

mysql

Code

This is the internal Docker hostname for the MySQL container.

---

### **Do I need to expose MySQL to the host?**
No.  
The database runs inside Docker and is accessed only by the CA application container.

---

## Configuration Questions

### **Why do I need to set `__CA_URL_ROOT__` manually?**
Auto‑detection fails in Docker because:

- CA is installed in subdirectories (`/ca`, `/capublic`)
- Apache uses aliases
- Docker changes the document root

Hard‑coding the correct URL root prevents redirect loops and routing errors.

---

### **Where do I set `__CA_URL_ROOT__`?**
In:

ca/app/post-setup.php
capublic/app/post-setup.php

Code

Providence:

```php
define("__CA_URL_ROOT__", "/ca");
Pawtucket:

php
define("__CA_URL_ROOT__", "/capublic");
Why does Pawtucket need a media symlink?
Because:

Providence stores all media

Pawtucket must read the same files

Pawtucket does not generate derivatives

CA’s media loader supports symlinks

The symlink ensures consistent media access.

Routing Questions
Why do I get an infinite redirect loop?
Cause:

incorrect __CA_URL_ROOT__

Fix:

php
define("__CA_URL_ROOT__", "/ca");
Why does Pawtucket load Providence pages?
Cause:

missing Apache alias

incorrect base directory

incorrect URL root

Fix:

verify /capublic alias

set correct URL root

Why do I get “Not Found” errors?
Cause:

Apache rewrite disabled

.htaccess ignored

Fix:

Code
a2enmod rewrite
AllowOverride All
Media Questions
Why are thumbnails broken?
Cause:

missing symlink

incorrect permissions

Fix:

Code
rm -rf capublic/media
ln -s ca/media capublic/media
chmod -R 775 ca/media
Where should I store media?
Always in:

Code
ca/media/
Pawtucket reads from the symlink.

Upgrade Questions
How do I upgrade CollectiveAccess?
Replace CA files

Run composer install

Restart containers

Clear caches

Visit /ca to apply migrations

Will I lose my database during upgrades?
No.
The database is stored in a Docker volume:

Code
ca-mysql-data
It persists across rebuilds.

Will I lose my media during upgrades?
No.
Media lives in the mounted directory:

Code
ca/media/
It is not inside the container filesystem.

Docker Questions
How do I rebuild the environment?
Code
docker-compose down
docker-compose build
docker-compose up -d
How do I access the CA container?
Code
docker exec -it ca-app bash
How do I back up the database?
Code
docker exec ca-mysql mysqldump -u root -proot collectiveaccess > backup.sql
Miscellaneous Questions
Can I use custom themes?
Yes.
Place them in:

Code
capublic/themes/<yourtheme>/
Can I use custom views or bundles?
Yes.
Place them in:

Code
ca/app/views/
ca/app/conf/bundles/
Can I run this on a NAS or cloud VM?
Yes.
Docker makes the environment portable across:

Synology

QNAP

Debian/Ubuntu servers

Windows WSL2

macOS

cloud VMs (AWS, Azure, GCP)

Summary
This FAQ covers the most common questions about installing, configuring, upgrading, and maintaining CollectiveAccess inside Docker. For deeper details, see the other documents in the /docs folder:

architecture.md

installation.md

configuration.md

routing.md

media-symlink.md

troubleshooting.md

upgrading.md
