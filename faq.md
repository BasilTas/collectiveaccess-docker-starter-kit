# Frequently Asked Questions (FAQ)

This document answers common questions about using the CollectiveAccess Docker Starter Kit. It covers installation, configuration, routing, media handling, MySQL behaviour, performance expectations, and general usage patterns.

---

## General Questions

### **What is this starter kit for?**
It provides a modern, reproducible Docker environment for running:

- Providence (backend)
- Pawtucket (frontend)
- MySQL 8.x
- PHP 8.x
- Apache with rewrite enabled

It includes fixes for routing, media handling, MySQL tuning, and configuration issues that commonly affect CA installations.

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

### **Why does the installer take ~800 seconds on Windows?**
Because Docker Desktop adds significant filesystem overhead.  
The CA installer is CPU‑bound and reads thousands of small PHP/XML files.

Typical install times:

- **Windows + Docker Desktop:** ~800 seconds  
- **WSL2:** ~300–400 seconds  
- **Native Linux:** ~200–300 seconds  

This is normal and not a sign of misconfiguration.

---

## Configuration Questions

### **Why is the MySQL buffer pool set using `command:` instead of a config file?**
Because MySQL config file mounts are unreliable on Windows.  
Some MySQL images silently ignore mounted config files.

The starter kit uses:

```yaml
command: --innodb-buffer-pool-size=1G
This method is:

reliable

cross‑platform

independent of Alpine/Debian differences

guaranteed to apply

Where do I set __CA_URL_ROOT__?
In:

Code
ca/app/conf/local/configuration.php
capublic/app/conf/local/configuration.php
Providence:

php
__CA_URL_ROOT__ = "/ca";
Pawtucket:

php
__CA_URL_ROOT__ = "/capublic";
These values match the Apache routing.

Why does Pawtucket need a media symlink?
Because:

Providence stores all media

Pawtucket must read the same files

Pawtucket does not generate derivatives

The symlink ensures consistent media access:

Code
capublic/media → ca/media
Routing Questions
Why do I get an infinite redirect loop?
Cause:

incorrect __CA_URL_ROOT__

Fix:

php
__CA_URL_ROOT__ = "/ca";
Why does Pawtucket load Providence pages?
Cause:

incorrect Apache alias

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

MySQL Questions
Why did my database disappear after switching MySQL images?
Because MySQL data directories are not portable between distributions.

If you switch images (Debian → Alpine or vice‑versa), you must delete the old volume:

Code
docker volume rm collectiveaccess-docker_mysql_data
This is normal and unavoidable.

How do I back up the database?
Code
docker exec ca-mysql mysqldump -u root -prootpass ca > backup.sql
How do I restore a backup?
Code
docker exec -i ca-mysql mysql -u root -prootpass ca < backup.sql
Docker Questions
How do I rebuild the environment?
Code
docker-compose down
docker-compose up -d --force-recreate
How do I access the CA container?
Code
docker exec -it ca-container bash
How do I access the MySQL container?
Code
docker exec -it ca-mysql bash
Performance Questions
Why does CA feel faster after tuning MySQL?
Because:

metadata loads stay hot

index pages stay in memory

first-page loads stop stalling

search performance improves

disk thrashing disappears

Runtime performance improves even though installer time does not.

Why is the installer still slow?
Because the installer is CPU‑bound and dominated by PHP parsing thousands of files.

MySQL tuning improves runtime performance, not installer duration.

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
