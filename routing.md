# Routing and URL Configuration

This document explains how routing works in the CollectiveAccess Docker Starter Kit. It covers Apache configuration, URL roots, container paths, and common routing issues such as redirect loops, broken assets, and incorrect media URLs.

The goal is to ensure Providence and Pawtucket load correctly under:

- `/ca` (Providence)
- `/capublic` (Pawtucket)

with predictable behaviour across Windows, macOS, and Linux.

---

## 1. Overview

The starter kit uses a single Apache + PHP container (`ca-app`) to serve both applications:

- Providence → `/var/www/html/ca`
- Pawtucket → `/var/www/html/capublic`

Routing is handled by:

- Apache configuration (`apache.conf`)
- CA’s `__CA_URL_ROOT__` overrides
- Docker port mapping (`8080 → Apache port 80`)

This ensures both applications are accessible without virtual hosts.

---

## 2. Apache Routing

The Apache configuration defines two application paths:

/ca        → /var/www/html/ca
/capublic  → /var/www/html/capublic

Code

Example excerpt:

```apache
Alias /ca /var/www/html/ca
Alias /capublic /var/www/html/capublic

<Directory /var/www/html/ca>
    AllowOverride All
    Require all granted
</Directory>

<Directory /var/www/html/capublic>
    AllowOverride All
    Require all granted
</Directory>
Rewrite rules are enabled via:

Code
a2enmod rewrite
This allows CA’s .htaccess files to function correctly.

3. URL Root Configuration
CollectiveAccess requires explicit URL root settings to avoid redirect loops and broken asset paths.

Providence (ca/app/conf/local/configuration.php)
php
__CA_URL_ROOT__ = "/ca";
Pawtucket (capublic/app/conf/local/configuration.php)
php
__CA_URL_ROOT__ = "/capublic";
These values must match the Apache aliases.

4. Docker Port Mapping
The starter kit exposes Apache on port 8080:

Code
http://localhost:8080/ca
http://localhost:8080/capublic
This avoids conflicts with other local web servers.

5. Media Routing
Pawtucket does not store its own media.
Instead, it uses a symlink:

Code
capublic/media → ca/media
This ensures:

thumbnails load correctly

zoomable images work

derivatives are consistent

no duplication of media files

See media-symlink.md for details.

6. Common Routing Problems and Fixes
Problem: Redirect loop when accessing Providence
Cause:

incorrect __CA_URL_ROOT__

Fix:

php
__CA_URL_ROOT__ = "/ca";
Problem: Pawtucket loads Providence pages
Cause:

incorrect Apache alias

incorrect URL root

Fix:

verify /capublic alias

set correct URL root

Problem: “Not Found” or missing CSS/JS
Cause:

Apache rewrite disabled

.htaccess ignored

Fix:
Ensure:

apache
AllowOverride All
is set for both /ca and /capublic.

Problem: Pawtucket thumbnails broken
Cause:

missing symlink

incorrect media path

Fix:
Recreate symlink:

Code
rm -rf capublic/media
ln -s /var/www/html/ca/media /var/www/html/capublic/media
Problem: Providence admin login redirects back to login
Cause:

incorrect URL root

missing rewrite rules

Fix:

verify __CA_URL_ROOT__

ensure rewrite module is enabled

7. Internal Routing Behaviour
Providence
uses /ca as its base path

loads assets relative to /ca

generates URLs using __CA_URL_ROOT__

Pawtucket
uses /capublic as its base path

loads media through the symlink

generates URLs using __CA_URL_ROOT__

Apache
maps both paths to the correct directories

handles rewrites for clean URLs

serves static assets directly

8. Windows Notes
When running under Docker Desktop on Windows:

routing behaves normally

rewrite rules work correctly

URL roots must be set explicitly

filesystem performance may affect installer speed, not routing

Routing is one of the most stable parts of the stack.

9. Summary
Correct routing requires:

Apache aliases for /ca and /capublic

rewrite rules enabled

correct __CA_URL_ROOT__ values

correct media symlink

correct Docker port mapping

With these elements in place, Providence and Pawtucket load reliably across all platforms.

See configuration.md for application settings and troubleshooting.md for routing-related issues.
