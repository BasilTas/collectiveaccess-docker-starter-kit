# CollectiveAccess Docker Architecture

This document describes the architecture of the CollectiveAccess Docker Starter Kit, including container layout, directory structure, networking, and how Providence and Pawtucket operate inside the Docker environment.

The goal of this starter kit is to provide a clean, predictable, and reproducible environment for running CollectiveAccess on modern PHP and MySQL versions, with sensible defaults and performance tuning applied.

---

## Overview

The starter kit provides a two‑container architecture:

- **ca-app** — Apache + PHP 8.x running both Providence and Pawtucket  
- **ca-mysql** — MySQL 8.x with tuned InnoDB settings

This layout keeps the stack simple while ensuring good performance and easy maintenance.

---

## Container Layout

### 1. `ca-app` (Apache + PHP + CollectiveAccess)

This container runs both applications:

- `/var/www/html/ca` → Providence (backend)
- `/var/www/html/capublic` → Pawtucket (frontend)

Shared components:

- `/var/www/html/ca/media` → Providence media directory  
- `/var/www/html/capublic/media` → symlink to Providence media  
- `php.ini` overrides for CA performance tuning  
- Apache configuration for routing and virtual paths

The container uses volume mounts so that CA files remain editable on the host.

### 2. `ca-mysql` (MySQL 8.x)

This container provides the database backend for both Providence and Pawtucket.

Key characteristics:

- Uses the **mysql/mysql-server:8.0** image  
- Stores data in a persistent Docker volume (`mysql_data`)  
- InnoDB buffer pool size is set via container `command:` for cross‑platform reliability  
- No external config files are mounted (Windows bind‑mounts can be unreliable)

The buffer pool is tuned to:

innodb_buffer_pool_size = 1G

Code

This dramatically improves CollectiveAccess runtime performance.

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

This mirrors the recommended CollectiveAccess layout.

---

## Networking

Docker Compose creates an internal network where:

- `ca-app` connects to MySQL using hostname `mysql`
- Ports exposed:
  - `8080` → Apache (Providence + Pawtucket)
- MySQL is not exposed publicly unless configured

This keeps the database isolated while allowing the web container full access.

---

## URL Routing

The starter kit uses simple, predictable paths:

- `http://localhost:8080/ca` → Providence  
- `http://localhost:8080/capublic` → Pawtucket  

Routing is controlled by:

- Apache configuration (`apache.conf`)
- CA’s `__CA_URL_ROOT__` overrides in `post-setup.php`

This avoids redirect loops and ensures correct asset paths.

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

This is the recommended CA deployment pattern.

---

## MySQL Performance Tuning

The MySQL container applies performance tuning via:

command: --innodb-buffer-pool-size=1G

Code

This approach is used because:

- MySQL config mounts are unreliable on Windows  
- Alpine/Debian images use different config directories  
- The buffer pool is critical for CA performance  
- The `command:` override works consistently across all platforms  

The tuned buffer pool significantly improves:

- metadata loading  
- search performance  
- first-page load latency  
- general responsiveness  

---

## Summary

This architecture provides:

- clean separation of application and database  
- shared media storage  
- modern PHP and MySQL versions  
- correct routing  
- reproducible Docker builds  
- reliable MySQL tuning  
- predictable behaviour on Windows, macOS, and Linux  

See other documents in `/docs` for installation, configuration, routing, and troubleshooting.
