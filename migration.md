# Migration Guide (Existing CollectiveAccess → Docker)

This document explains how to migrate an existing CollectiveAccess installation (Providence + Pawtucket) into the Docker Starter Kit. It covers the IIS-style upgrade workflow, what to copy, what not to copy, how to import your database, and how to trigger CA’s built‑in upgrade process.

If you are upgrading an existing Docker installation to a newer CA version, see upgrading.md.

This guide is for users who already have:

- a working CA installation  
- an existing MySQL database  
- existing media  
- existing themes, bundles, or views  
- existing `setup.php` and `post-setup.php`  

If you are performing a **fresh install**, see `installation.md` instead.

---

# 1. Overview: Two Types of Installation

There are two fundamentally different ways to use this starter kit:

### **A. Fresh Install**
You run the Providence installer inside Docker.  
This creates a new database, new admin account, and new configuration files.

### **B. Migration (this document)**
You already have CA running elsewhere (IIS, Apache, Linux, macOS).  
You bring your existing:

- database  
- media  
- configuration  
- customisations  

into Docker.

This guide explains Scenario B.

---

# 2. What You MUST Copy from Your Existing Installation

From your existing CA installation, copy:

### ✔ Providence application directory
ca/

Code

### ✔ Pawtucket application directory
capublic/

Code

### ✔ Media directory
ca/media/

Code

This is critical — it contains originals and derivatives.

### ✔ Configuration files
ca/setup.php
ca/post-setup.php
capublic/setup.php
capublic/post-setup.php

Code

### ✔ Custom themes
capublic/themes/<yourtheme>/

Code

### ✔ Custom bundles, views, or local overrides
ca/app/conf/bundles/
ca/app/views/
capublic/app/conf/
capublic/app/views/

Code

### ✔ Any local configuration files
(e.g., `local.conf`, custom plugin configs)

---

# 3. What You MUST NOT Copy

Do **not** copy:

- old PHP binaries  
- old Apache configs  
- old MySQL binaries  
- old `.htaccess` files if they conflict with Docker’s routing  
- cache directories (`app/tmp`)  
- log directories  

These will be regenerated automatically.

---

# 4. Prepare Your Docker Environment

Start the Docker environment:

docker-compose up -d

Code

This creates:

- `ca-app` (Apache + PHP)
- `ca-mysql` (MySQL 8.x)

Stop the containers before replacing files:

docker-compose down

Code

---

# 5. Import Your Existing MySQL Database

Export your existing database from IIS/Linux/macOS:

mysqldump -u root -p collectiveaccess > backup.sql

Code

Import it into Docker:

docker exec -i ca-mysql mysql -u root -proot collectiveaccess < backup.sql

Code

Your users, passwords, roles, metadata, and objects are now inside Docker.

---

# 6. Replace the Application Directories

You now replace the CA application files inside Docker with your existing ones.

If your repo bind‑mounts the directories (recommended), simply copy your existing CA directories into:

docker/ca/
docker/capublic/

Code

If you prefer replacing inside the container:

docker exec -it ca-app bash
cd /var/www/html

mv ca ca_old
mv capublic capublic_old

cp -R /path/to/your/ca /var/www/html/ca
cp -R /path/to/your/capublic /var/www/html/capublic

Code

---

# 7. Apply Docker-Specific Fixes

Your existing CA installation was configured for IIS or bare‑metal Apache.  
Docker requires a few adjustments.

### ✔ Update MySQL hostname
In both `setup.php` files:

define("CA_DB_HOST", "mysql");

Code

### ✔ Update URL roots
Providence:

```php
define("__CA_URL_ROOT__", "/ca");
Pawtucket:

php
define("__CA_URL_ROOT__", "/capublic");
✔ Recreate media symlink
Inside container:

Code
rm -rf /var/www/html/capublic/media
ln -s /var/www/html/ca/media /var/www/html/capublic/media
✔ Fix permissions
Code
chmod -R 775 /var/www/html/ca/app/tmp
chmod -R 775 /var/www/html/capublic/app/tmp
chown -R www-data:www-data /var/www/html/ca/app/tmp
chown -R www-data:www-data /var/www/html/capublic/app/tmp
✔ Ensure Apache rewrite is enabled
The Dockerfile already does this.

8. Restart the Application
Start the containers again:

Code
docker-compose up -d
9. Trigger the CollectiveAccess Upgrade Process
Visit Providence:

Code
http://localhost:8080/ca
If your existing database schema is older than the CA version you copied:

CA will detect the version change

CA will prompt you to run the upgrade

CA will apply migrations automatically

This is identical to the IIS workflow:

old directories renamed

new directories put in place

CA upgrades the database

users and passwords remain intact

10. Verify the Migration
✔ Providence loads
✔ Pawtucket loads
✔ Media loads
✔ Login works
✔ Browse/search works
✔ Custom themes load
✔ No redirect loops
✔ No missing derivatives
If anything fails, see troubleshooting.md.

11. Rollback Strategy
If something goes wrong:

Stop containers

Restore your old CA directories

Restore your old MySQL dump

Start containers again

Docker makes rollback trivial because:

the database is in a volume

the application files are bind‑mounted

nothing is overwritten permanently

Summary
Migrating an existing CA installation into Docker is straightforward:

copy your existing CA directories

import your existing database

apply Docker-specific routing fixes

recreate the media symlink

restart containers

let CA run its upgrade process

This workflow mirrors the IIS upgrade method and preserves all users, passwords, media, and customisations.

For fresh installs, see installation.md.
For routing details, see routing.md.
For media handling, see media-symlink.md.
For upgrade details, see upgrading.md.
