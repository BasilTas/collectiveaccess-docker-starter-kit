architecture.md
markdown
# CollectiveAccess Docker Architecture

This document explains the overall architecture of the CollectiveAccess Docker Starter Kit, including container layout, directory structure, networking, and how Providence and Pawtucket interact inside the Docker environment.

---

## Overview

This starter kit provides a clean, modern Docker environment for:

- **Providence** (administrative backend)
- **Pawtucket** (public frontend)
- **MySQL 8.x** (database)
- **Apache + PHP 8.4** (application server)

The architecture is designed to be simple, reproducible, and easy to maintain.

---

## Container Layout

### 1. `ca-app` (Apache + PHP + CA)
Runs both Providence and Pawtucket.

Contents:
- `/var/www/html/ca` → Providence
- `/var/www/html/capublic` → Pawtucket
- `/var/www/html/ca/media` → shared media directory
- `/var/www/html/capublic/media` → symlink to Providence media

### 2. `ca-mysql` (MySQL 8.x)
Stores all CA data:
- objects
- entities
- relationships
- metadata
- configuration
- logs

Accessible inside Docker at hostname:
mysql

Code

---

## Directory Structure

Inside the `ca-app` container:

/var/www/html/
ca/               ← Providence
app/
media/
themes/
vendor/
setup.php
post-setup.php

capublic/         ← Pawtucket
app/
media → symlink to /var/www/html/ca/media
themes/
vendor/
setup.php
post-setup.php

Code

This mirrors the recommended CA layout used in many production environments.

---

## Networking

Docker Compose creates an internal network:

- `ca-app` connects to MySQL using hostname `mysql`
- Ports exposed:
  - `8080` → Apache (Providence + Pawtucket)
  - MySQL is **not** exposed publicly unless configured

---

## URL Routing

The starter kit uses:

- `http://localhost:8080/ca` → Providence
- `http://localhost:8080/capublic` → Pawtucket

These paths are defined by:

- Apache alias rules
- `__CA_URL_ROOT__` overrides in `post-setup.php`

This avoids redirect loops and ensures correct routing.

---

## Media Handling

Pawtucket does not store its own media.  
Instead, it uses a symlink:

capublic/media → ca/media

Code

This ensures:
- no duplication
- consistent derivatives
- correct public access
- correct backend access

---

## Summary

This architecture provides:

- clean separation of Providence and Pawtucket
- shared media storage
- modern PHP and MySQL versions
- correct routing
- reproducible Docker builds
- easy migration from IIS or other environments

See other documents in `/docs` for installation, configuration, routing, and troubleshooting.
