# Upgrading CollectiveAccess, PHP, and MySQL

This document explains how to safely upgrade CollectiveAccess, PHP, MySQL, and related components inside the Docker Starter Kit. It covers version compatibility, recommended upgrade paths, and how to rebuild containers without losing data.

---

## Overview

Upgrading in Docker is simpler than traditional installations because:

- the application environment is defined by the Dockerfile  
- the database persists in a volume  
- containers can be rebuilt cleanly  
- configuration files remain untouched  

This guide covers upgrades for:

- CollectiveAccess (Providence + Pawtucket)
- PHP
- MySQL
- Composer dependencies
- Docker containers

---

# 1. Upgrading CollectiveAccess

## Step 1 — Download the new CA release

Download the latest Providence and Pawtucket versions from:

https://github.com/collectiveaccess (github.com in Bing)

Code

Replace the contents of:

/var/www/html/ca
/var/www/html/capublic

Code

while preserving:

- `setup.php`
- `post-setup.php`
- `media/` directory
- `app/tmp/` directory
- custom themes
- custom views
- custom configuration files

---

## Step 2 — Reinstall vendor dependencies

Inside the container:

docker exec -it ca-app bash
cd /var/www/html/ca
composer install

cd /var/www/html/capublic
composer install

Code

This ensures compatibility with PHP 8.4 and modern CA releases.

---

## Step 3 — Run database migrations

Providence automatically applies migrations on first load.

Visit:

http://localhost:8080/ca

Code

If migrations are required, CA will prompt you.

---

## Step 4 — Clear caches

Inside the container:

rm -rf /var/www/html/ca/app/tmp/*
rm -rf /var/www/html/capublic/app/tmp/*

Code

---

# 2. Upgrading PHP

The PHP version is defined in the Dockerfile:

FROM php:8.4-apache

Code

To upgrade:

1. Edit the Dockerfile to the desired PHP version  
2. Rebuild the container:

docker-compose build --no-cache
docker-compose up -d

Code

### Notes

- CA supports modern PHP versions (8.1–8.4)  
- Always run `composer install` after upgrading PHP  
- Some extensions may require updates  

---

# 3. Upgrading MySQL

The MySQL version is defined in `docker-compose.yml`:

image: mysql:8

Code

To upgrade:

1. Change the version (e.g., `mysql:8.4`)  
2. Stop containers:

docker-compose down

Code

3. Start containers:

docker-compose up -d

Code

### Important

Your database is stored in a Docker volume:

ca-mysql-data

Code

This volume is **not deleted** unless you explicitly remove it.

If MySQL performs an internal upgrade, it will do so automatically on startup.

---

# 4. Upgrading Composer Dependencies

Inside the container:

docker exec -it ca-app bash
cd /var/www/html/ca
composer update

cd /var/www/html/capublic
composer update

Code

### When to update

- after upgrading PHP  
- after upgrading CA  
- when security patches are released  

---

# 5. Rebuilding Containers Safely

To rebuild without losing data:

docker-compose down
docker-compose build
docker-compose up -d

Code

Your database persists because it lives in a volume.

Your media persists because it lives in the mounted directory.

Your configuration persists because it lives in the repo.

---

# 6. Backing Up Before Upgrading

### Database backup

docker exec ca-mysql mysqldump -u root -proot collectiveaccess > backup.sql

Code

### Media backup

Copy:

ca/media/

Code

### Configuration backup

Copy:

setup.php
post-setup.php
themes/
views/

Code

---

# 7. Common Upgrade Issues

### Providence fails to load after upgrade
Cause:
- missing vendor dependencies

Fix:
composer install

Code

### Pawtucket shows blank pages
Cause:
- missing theme files
- missing symlink

Fix:
- restore theme
- recreate symlink

### MySQL refuses connection
Cause:
- version mismatch

Fix:
- verify `mysql` hostname
- check volume compatibility

---

# Summary

Upgrading in Docker is safe and predictable:

- rebuild containers anytime  
- database persists  
- media persists  
- configuration persists  
- CA migrations run automatically  

Follow this guide to upgrade CA, PHP, MySQL, and dependencies without downtime or data loss.

See `faq.md` for additional upgrade-related questions.
