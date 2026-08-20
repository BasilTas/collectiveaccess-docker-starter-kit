# Migrating an Existing CollectiveAccess Installation

This document explains how to migrate an existing CollectiveAccess installation (Providence + Pawtucket) into the Docker Starter Kit. It covers database export/import, media transfer, configuration alignment, and common pitfalls when moving from IIS, Apache, or older Linux systems.

The goal is to provide a predictable, safe migration path that avoids data loss and ensures the Docker environment behaves identically to your previous installation.

---

## 1. What You Can Migrate

You can migrate:

- Providence database  
- Pawtucket database (if separate)  
- Media directory (`media/`)  
- Themes  
- Views and bundles  
- Custom configuration files  
- Plugins  
- Profile XML files  

You **cannot** migrate:

- MySQL data directories  
- Apache/IIS configuration files  
- PHP runtime settings  
- Old MySQL binary logs  

These must be recreated inside Docker.

---

## 2. Export Your Existing Database

From your old server (IIS, Linux, macOS, or WAMP/XAMPP):

mysqldump -u <user> -p <database> > ca_backup.sql

Code

Recommended flags for large CA databases:

mysqldump --single-transaction --routines --events --default-character-set=utf8mb4 \
-u <user> -p <database> > ca_backup.sql

Code

This produces a portable SQL dump compatible with MySQL 8.x.

---

## 3. Copy the Backup Into the Project Directory

Place your SQL file in the root of the starter kit:

collectiveaccess-docker/
docker-compose.yml
ca/
capublic/
ca_backup.sql   ← here

Code

---

## 4. Start the Docker Environment

docker-compose up -d

Code

This launches:

- `ca-app` (Apache + PHP + CA)
- `ca-mysql` (MySQL 8.x)

---

## 5. Import the Database Into Docker

Run:

docker exec -i ca-mysql mysql -u root -prootpass ca < ca_backup.sql

Code

This restores all tables, metadata, and records.

If your old installation used a different database name, adjust accordingly.

---

## 6. Copy Your Media Directory

Your old installation has a directory like:

/path/to/old/ca/media/

Code

Copy it into the new CA directory:

collectiveaccess-docker/ca/media/

Code

If the directory already exists, replace it.

---

## 7. Verify the Pawtucket Symlink

Inside the CA container:

docker exec -it ca-app bash
ls -l /var/www/html/capublic/media

Code

Expected:

media -> /var/www/html/ca/media

Code

If missing:

rm -rf /var/www/html/capublic/media
ln -s /var/www/html/ca/media /var/www/html/capublic/media

Code

---

## 8. Update Configuration Files

Your old installation may have custom settings in:

- `app/conf/local/configuration.php`
- `app/conf/local/app.conf`
- `app/conf/local/search.conf`
- `app/conf/local/media_processing.conf`

Copy these into:

ca/app/conf/local/
capublic/app/conf/local/

Code

Then update:

### Database host
CA_DB_HOST = "mysql";

Code

### URL roots
Providence:
CA_URL_ROOT = "/ca";

Code

Pawtucket:
CA_URL_ROOT = "/capublic";

Code

### Media directory
Should remain:
CA_MEDIA_ROOT = "/var/www/html/ca/media";

Code

---

## 9. Restart the Containers

docker-compose down
docker-compose up -d

Code

This reloads configuration and ensures CA sees the restored database.

---

## 10. Log In and Verify

Providence:

http://localhost:8080/ca

Code

Pawtucket:

http://localhost:8080/capublic

Code

Check:

- media loads  
- thumbnails appear  
- search works  
- records display correctly  
- themes load  
- plugins behave normally  

---

## 11. Important Notes About MySQL Migration

### MySQL data directories cannot be migrated
You **must not** copy your old MySQL data directory into Docker.  
MySQL data directories are not portable between:

- versions  
- distributions  
- operating systems  
- storage engines  

This will cause:

- `binlog.index` errors  
- permission failures  
- corrupted tables  
- MySQL refusing to start  

Always migrate using **SQL dumps**, never raw data directories.

---

### Switching MySQL images requires deleting the volume
If you change the MySQL image (Debian → Alpine or vice‑versa):

docker volume rm collectiveaccess-docker_mysql_data

Code

This is normal and unavoidable.

---

### Buffer pool tuning is handled via `command:`
The starter kit uses:

command: --innodb-buffer-pool-size=1G

Code

This ensures consistent performance across Windows, macOS, and Linux.

---

## 12. Migrating Pawtucket Themes

Copy your custom theme:

old_install/capublic/themes/<yourtheme>/

Code

into:

collectiveaccess-docker/capublic/themes/<yourtheme>/

Code

Then update:

capublic/app/conf/local/app.conf

Code

Set:

theme = "<yourtheme>"

Code

---

## 13. Migrating Providence Customizations

Copy:

- custom views  
- custom bundles  
- custom display templates  
- custom profile XML  
- custom plugins  

into the corresponding directories under:

collectiveaccess-docker/ca/

Code

Restart the container afterwards.

---

## 14. Summary

Migrating to the Docker Starter Kit involves:

1. Exporting your old database  
2. Importing it into the Docker MySQL container  
3. Copying your media directory  
4. Copying custom configuration files  
5. Updating URL roots and database hostnames  
6. Restarting the environment  
7. Verifying Providence and Pawtucket  

This process produces a clean, modern, reproducible CA environment with improved performance and easier maintenance.

See `upgrading.md` for version‑to‑version upgrade guidance.
