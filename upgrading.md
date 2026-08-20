# Upgrading CollectiveAccess in Docker

This document explains how to upgrade a CollectiveAccess installation running inside the Docker Starter Kit. It covers upgrading Providence, Pawtucket, MySQL, PHP, and the Docker environment itself, while preserving your database, media, and customizations.

The goal is to provide a safe, predictable upgrade workflow that avoids data loss and ensures the new version behaves identically to the old one.

---

# 1. What an Upgrade Involves

Upgrading CollectiveAccess inside Docker requires:

- replacing CA application files  
- preserving configuration files  
- preserving media  
- preserving custom themes, bundles, and views  
- keeping the existing MySQL data volume  
- allowing CA to run its internal database migrations  

This workflow is similar to upgrading CA on a traditional server, but simplified by Docker’s reproducibility.

---

# 2. Files You MUST Preserve

These files and directories must be kept during any upgrade:

ca/setup.php
ca/post-setup.php
ca/media/
ca/app/conf/local/
ca/themes/ (if custom)
ca/app/conf/bundles/ (if custom)
ca/app/views/ (if custom)

capublic/setup.php
capublic/post-setup.php
capublic/app/conf/local/
capublic/themes/ (if custom)

Code

These contain:

- database credentials  
- URL root overrides  
- media  
- customizations  
- local configuration  
- theme settings  

Do **not** overwrite these during an upgrade.

---

# 3. Files You MUST Replace

Replace the following with the new version:

ca/app/
ca/vendor/
ca/themes/ (default themes only)
capublic/app/
capublic/vendor/
capublic/themes/ (default themes only)

Code

These contain the CA core codebase.

---

# 4. Files You MUST NOT Copy

Do **not** copy:

ca/app/tmp/
ca/app/log/
capublic/app/tmp/
capublic/app/log/
.htaccess
old Apache/IIS configs
old PHP configs

Code

These are environment‑specific and should be regenerated.

---

# 5. Upgrade Workflow

## Step 1 — Stop the environment

docker-compose down

Code

This preserves the MySQL volume.

---

## Step 2 — Replace CA application files

Extract the new CA version and copy:

- `app/`
- `vendor/`
- default themes

into:

collectiveaccess-docker/ca/
collectiveaccess-docker/capublic/

Code

Do **not** overwrite:

- `setup.php`
- `post-setup.php`
- `media/`
- `app/conf/local/`
- custom themes  
- custom bundles/views  

---

## Step 3 — Verify the Pawtucket media symlink

Inside the container:

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

## Step 4 — Start the environment

docker-compose up -d

Code

---

## Step 5 — Allow CA to run database migrations

Visit:

http://localhost:8080/ca

Code

If CA detects a version change, it will:

- prompt for database upgrade  
- apply schema migrations  
- update metadata  
- rebuild caches  

This process is automatic.

---

# 6. Upgrading MySQL

## Important: MySQL data directories are NOT portable

You **cannot** upgrade MySQL by switching images without deleting the data volume.

If you switch MySQL images (Debian ↔ Alpine):

docker volume rm collectiveaccess-docker_mysql_data

Code

This wipes the database.

### Safe MySQL upgrade path

1. Export your database:
docker exec ca-mysql mysqldump -u root -prootpass ca > backup.sql

Code

2. Change the MySQL image in `docker-compose.yml`

3. Delete the old volume:
docker volume rm collectiveaccess-docker_mysql_data

Code

4. Start the new MySQL container:
docker-compose up -d

Code

5. Import your backup:
docker exec -i ca-mysql mysql -u root -prootpass ca < backup.sql

Code

This is the only safe way to upgrade MySQL.

---

# 7. Upgrading PHP or Apache

To upgrade PHP or Apache:

1. Edit the Dockerfile to use a newer base image  
2. Rebuild the CA container:
docker-compose build --no-cache
docker-compose up -d

Code

Your CA files and MySQL data remain intact.

---

# 8. Upgrading the Docker Starter Kit Itself

If the starter kit receives updates:

- new `docker-compose.yml`
- new `php.ini`
- new `apache.conf`
- new documentation

You may safely replace these files **except**:

- your CA application directories  
- your MySQL volume  
- your media directory  

---

# 9. Common Upgrade Problems

## 9.1 CA reports “database not installed”
Cause:
- MySQL volume was deleted  
- MySQL failed to start  
- wrong database host

Fix:
- ensure MySQL is running  
- use hostname `mysql`  
- re-import backup if needed  

---

## 9.2 MySQL fails after switching images
Cause:
- incompatible data directory

Fix:
docker volume rm collectiveaccess-docker_mysql_data

Code

---

## 9.3 Pawtucket thumbnails broken
Cause:
- symlink missing

Fix:
ln -s /var/www/html/ca/media /var/www/html/capublic/media

Code

---

## 9.4 Redirect loops after upgrade
Cause:
- overwritten `configuration.php`

Fix:
Providence:
```php
__CA_URL_ROOT__ = "/ca";
Pawtucket:

php
__CA_URL_ROOT__ = "/capublic";
10. Summary
Upgrading CA inside Docker requires:

preserving configuration and media

replacing CA core files

keeping the MySQL volume

allowing CA to run migrations

avoiding raw MySQL directory migration

using SQL dumps for MySQL upgrades

verifying routing and symlinks

This workflow ensures safe, predictable upgrades across all platforms.

See migrating.md for migration from external servers.
