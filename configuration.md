# Configuration Guide

This document explains the essential configuration steps required to run CollectiveAccess (Providence + Pawtucket) correctly inside the Docker Starter Kit. It covers PHP settings, URL root overrides, permissions, database connectivity, and recommended adjustments to CA’s setup files.

---

## Overview

CollectiveAccess requires several configuration adjustments when running inside Docker:

- Correct PHP settings
- Correct database hostname
- Correct URL routing
- Correct media directory handling
- Correct permissions for temporary directories
- Correct overrides in `setup.php` and `post-setup.php`

This guide documents all required changes.

---

## 1. PHP Configuration (`php.ini`)

The starter kit includes a tuned `php.ini` file with recommended settings:

### Memory and execution
memory_limit = 512M
max_execution_time = 120

Code

### Upload limits
upload_max_filesize = 64M
post_max_size = 64M

Code

### Timezone
date.timezone = UTC

Code

### Error visibility
display_errors = Off
log_errors = On

Code

These settings ensure CA runs smoothly and avoids common upload or timeout issues.

---

## 2. Database Configuration

Inside Docker, the MySQL hostname is:

mysql

Code

This must be used during installation and in CA’s configuration files.

### Installer settings:

- Host: `mysql`
- Database: `collectiveaccess`
- Username: `root`
- Password: `root`

These values match the `docker-compose.yml` configuration.

---

## 3. Providence Configuration

Providence lives at:

/var/www/html/ca

Code

### Required override in `post-setup.php`

Replace the auto-generated URL root with:

```php
define("__CA_URL_ROOT__", "/ca");
This prevents infinite redirect loops and ensures correct routing.

Optional override in setup.php
Ensure:

php
define("__CA_BASE_DIR__", "/var/www/html/ca");
4. Pawtucket Configuration
Pawtucket lives at:

Code
/var/www/html/capublic
Required override in post-setup.php
Replace the auto-generated URL root with:

php
define("__CA_URL_ROOT__", "/capublic");
This ensures Pawtucket builds correct URLs for:

browse pages

detail pages

media

search

login redirects

Optional override in setup.php
Ensure:

php
define("__CA_BASE_DIR__", "/var/www/html/capublic");
5. Media Directory Symlink
Pawtucket must use Providence’s media directory.

Inside the container:

Code
rm -rf /var/www/html/capublic/media
ln -s /var/www/html/ca/media /var/www/html/capublic/media
Verify:

Code
ls -l /var/www/html/capublic/media
Expected:

Code
media -> /var/www/html/ca/media
This ensures:

no duplication

correct derivatives

correct public access

correct backend access

6. Permissions
CA requires writable temporary directories.

Inside the container:

Code
chmod -R 775 /var/www/html/ca/app/tmp
chmod -R 775 /var/www/html/capublic/app/tmp

chown -R www-data:www-data /var/www/html/ca/app/tmp
chown -R www-data:www-data /var/www/html/capublic/app/tmp
Without these permissions, CA may fail silently.

7. Apache Rewrite Configuration
The Dockerfile enables rewrite:

Code
a2enmod rewrite
The Apache site configuration must allow .htaccess:

Code
<Directory /var/www/html>
    AllowOverride All
    Require all granted
</Directory>
This is required for CA’s routing system.

8. Verifying Configuration
Providence
Visit:

Code
http://localhost:8080/ca
If login works without redirect loops, configuration is correct.

Pawtucket
Visit:

Code
http://localhost:8080/capublic
If media loads and pages route correctly, configuration is correct.

Summary
This configuration ensures:

correct routing for Providence and Pawtucket

correct media handling

correct database connectivity

correct PHP environment

correct permissions

stable, reproducible Docker behaviour

Proceed to routing.md for deeper details on URL handling and rewrite rules.
