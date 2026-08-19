# Troubleshooting Guide

This document lists common issues encountered when running CollectiveAccess (Providence + Pawtucket) inside the Docker Starter Kit, along with clear explanations and fixes. These solutions are based on real-world deployment experience and address problems frequently seen in Docker, Apache, and CA routing.

---

## Overview

Most issues fall into a few categories:

- Incorrect URL routing
- Incorrect `__CA_URL_ROOT__`
- Missing media symlink
- Incorrect permissions
- Incorrect MySQL hostname
- Apache rewrite not enabled
- Incorrect directory layout

This guide provides quick diagnosis and fixes for each.

---

# 1. Infinite Redirect Loop (Providence)

### Symptoms
- Visiting `/ca` redirects repeatedly
- Login page never appears
- Browser shows “too many redirects”

### Cause
Incorrect `__CA_URL_ROOT__` in `post-setup.php`.

### Fix
Set:

```php
define("__CA_URL_ROOT__", "/ca");
Restart container:

Code
docker-compose restart ca-app
2. Pawtucket Loads Providence Pages
Symptoms
Visiting /capublic shows Providence login

Public pages missing

Pawtucket theme not loading

Cause
Incorrect base directory or missing Apache alias.

Fix
Verify:

Code
Alias /capublic /var/www/html/capublic
And in post-setup.php:

php
define("__CA_URL_ROOT__", "/capublic");
3. Media Not Loading (Broken Thumbnails)
Symptoms
Pawtucket shows broken images

Providence shows missing derivatives

Media viewers fail

Cause
Missing or incorrect symlink.

Fix
Inside container:

Code
rm -rf /var/www/html/capublic/media
ln -s /var/www/html/ca/media /var/www/html/capublic/media
Verify:

Code
ls -l /var/www/html/capublic/media
Expected:

Code
media -> /var/www/html/ca/media
4. “Media Not Found” Errors
Symptoms
Pawtucket pages load but media does not

Providence shows missing media paths

Causes
Incorrect URL root

Incorrect routing

Permissions issue

Fixes
Verify __CA_URL_ROOT__

Verify symlink

Fix permissions:

Code
chmod -R 775 /var/www/html/ca/media
chown -R www-data:www-data /var/www/html/ca/media
5. Installer Cannot Connect to Database
Symptoms
Installer says “Cannot connect to database”

MySQL connection refused

Cause
Incorrect hostname.

Fix
Use:

Code
mysql
Not:

localhost

127.0.0.1

container IP

These do not work inside Docker.

6. “Not Found” Errors for Pages
Symptoms
Visiting /ca or /capublic shows 404

Internal pages fail

Controllers not found

Cause
Apache rewrite disabled.

Fix
Rewrite module must be enabled:

Code
a2enmod rewrite
And Apache must allow .htaccess:

Code
<Directory /var/www/html>
    AllowOverride All
</Directory>
7. Pawtucket Theme Not Loading
Symptoms
Pawtucket loads but looks unstyled

Missing CSS, JS, images

Cause
Incorrect URL root or missing theme path.

Fix
Verify:

Code
__CA_URL_ROOT__ = "/capublic"
And theme directory exists:

Code
capublic/themes/<yourtheme>/
8. Providence Cannot Write Temporary Files
Symptoms
Installer fails

Derivatives not generated

Logs missing

Cause
Permissions on app/tmp.

Fix
Code
chmod -R 775 /var/www/html/ca/app/tmp
chmod -R 775 /var/www/html/capublic/app/tmp

chown -R www-data:www-data /var/www/html/ca/app/tmp
chown -R www-data:www-data /var/www/html/capublic/app/tmp
9. Pawtucket Search Returns No Results
Symptoms
Search page loads but shows nothing

Browse pages empty

Causes
Incorrect search configuration

Missing index

Incorrect URL root

Fixes
Verify /capublic routing

Rebuild search index in Providence

Check search.conf in Pawtucket

10. Providence Installer Stuck or Repeating
Symptoms
Installer loops

Installer restarts unexpectedly

Cause
Incorrect routing or missing rewrite.

Fix
Verify:

rewrite enabled

/ca/install reachable

__CA_URL_ROOT__ = "/ca"

Summary
Most issues are caused by:

incorrect URL roots

missing symlink

incorrect permissions

missing rewrite module

incorrect MySQL hostname

This guide provides fixes for all common problems encountered when deploying CA in Docker.

See upgrading.md for guidance on updating CA, PHP, or MySQL.
