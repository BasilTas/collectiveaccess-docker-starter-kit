# Routing and URL Configuration

This document explains how URL routing works in the CollectiveAccess Docker Starter Kit. It covers Apache rewrite rules, directory aliases, URL root overrides, and the fixes required to prevent redirect loops in Providence and Pawtucket.

---

## Overview

CollectiveAccess uses a custom routing system that depends on:

- Apache rewrite rules
- Correct directory structure
- Correct `__CA_URL_ROOT__` values
- Correct base directory definitions
- Correct `.htaccess` behaviour

In Docker, the default auto-detection often fails, causing:

- infinite redirect loops
- broken login redirects
- missing media
- incorrect controller paths
- “Not Found” errors

This guide documents the correct routing configuration.

---

## 1. URL Structure

The starter kit uses a clean, predictable URL layout:

### Providence (backend)
http://localhost:8080/ca

Code

### Pawtucket (frontend)
http://localhost:8080/capublic

Code

These paths are created using Apache aliases and CA configuration overrides.

---

## 2. Apache Configuration

Apache must allow `.htaccess` files to control routing.

### Required configuration:

<Directory /var/www/html>
AllowOverride All
Require all granted
</Directory>

Code

This enables:

- URL rewriting
- CA’s internal routing system
- clean URLs (no index.php)
- controller/action mapping

The Dockerfile automatically enables the rewrite module:

a2enmod rewrite

Code

---

## 3. Providence Routing

Providence lives at:

/var/www/html/ca

Code

### Required override in `post-setup.php`

Replace the auto-generated value with:

```php
define("__CA_URL_ROOT__", "/ca");
This ensures Providence builds correct URLs for:

login

logout

object detail pages

browse pages

media access

controller routing

Without this override, Providence may redirect to / or /index.php, causing loops.

4. Pawtucket Routing
Pawtucket lives at:

Code
/var/www/html/capublic
Required override in post-setup.php
Replace the auto-generated value with:

php
define("__CA_URL_ROOT__", "/capublic");
This ensures Pawtucket builds correct URLs for:

public browse pages

public detail pages

media viewers

search results

theme assets

Without this override, Pawtucket may attempt to load pages from / or /ca.

5. Why Auto-Detection Fails in Docker
CA attempts to compute the URL root using:

php
str_replace($_SERVER['DOCUMENT_ROOT'], '', __CA_BASE_DIR__);
This fails when:

CA is installed in a subdirectory

Apache uses an alias

Docker changes the document root

the installer runs under a different context

the container uses a virtual host

The result is often:

Code
__CA_URL_ROOT__ = ""
Which causes:

infinite redirects

broken login

missing assets

incorrect controller paths

Hard-coding the correct URL root fixes all of these issues.

6. Directory Aliases
The starter kit uses Apache aliases to map URLs to directories:

Code
Alias /ca /var/www/html/ca
Alias /capublic /var/www/html/capublic
These ensure:

Providence loads from /ca

Pawtucket loads from /capublic

both applications coexist cleanly

media symlink works correctly

7. Testing Routing
Providence
Visit:

Code
http://localhost:8080/ca
Expected:

login page loads

no redirect loop

URLs contain /ca/...

Pawtucket
Visit:

Code
http://localhost:8080/capublic
Expected:

homepage loads

media thumbnails appear

URLs contain /capublic/...

8. Common Routing Problems
Infinite redirect loop
Cause:

incorrect __CA_URL_ROOT__

Fix:

set /ca or /capublic explicitly

Pawtucket loads Providence pages
Cause:

missing alias

incorrect base directory

Fix:

verify Apache alias configuration

Media not loading
Cause:

missing symlink

incorrect URL root

Fix:

recreate symlink

verify /capublic routing

“Not Found” errors
Cause:

rewrite disabled

Fix:

ensure AllowOverride All is set

Summary
Correct routing requires:

Apache rewrite enabled

directory aliases for /ca and /capublic

hard-coded __CA_URL_ROOT__ values

correct base directory definitions

correct media symlink

With these settings, Providence and Pawtucket run cleanly inside Docker without redirect loops or routing errors.

See media-symlink.md for details on shared media handling.
