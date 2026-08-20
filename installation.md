# Installation Guide

This document explains how to install CollectiveAccess (Providence + Pawtucket) using the Docker Starter Kit. It covers container startup, running the installer, creating the database, and verifying the environment.

The goal is to provide a clean, predictable installation workflow that works reliably across Windows, macOS, and Linux.

---

## 1. Prerequisites

Before you begin, ensure you have:

- Docker Desktop (Windows/macOS) or Docker Engine (Linux)
- Docker Compose
- A web browser

No PHP, MySQL, or Apache installation is required on the host system.

---

## 2. Start the Containers

From the project root:

docker-compose up -d

Code

This launches:

- **ca-app** (Apache + PHP + Providence + Pawtucket)
- **ca-mysql** (MySQL 8.x with tuned buffer pool)

You can verify they are running:

docker ps

Code

Expected:

- `ca-app` → Up  
- `ca-mysql` → Up  

---

## 3. Access the Providence Installer

Open your browser:

http://localhost:8080/ca/install

Code

If you see the installer screen, your environment is working.

---

## 4. Database Configuration

During installation, use the following settings:

**Database host:**
mysql

Code

**Database name:**
ca

Code

**Username:**
causer

Code

**Password:**
capass

Code

These values match the `docker-compose.yml` configuration.

---

## 5. Complete the Installer

The installer will:

- create all required tables  
- populate initial configuration  
- generate default roles  
- create the admin user  
- verify media directories  
- verify PHP extensions  

This process may take **~800 seconds on Windows** due to Docker Desktop filesystem overhead.  
This is normal and not a sign of misconfiguration.

Typical install times:

- Windows + Docker Desktop: ~800 seconds  
- WSL2: ~300–400 seconds  
- Native Linux: ~200–300 seconds  

---

## 6. Log In to Providence

Visit:

http://localhost:8080/ca

Code

Enter the admin credentials you created during installation.

If login succeeds, Providence is fully operational.

---

## 7. Access Pawtucket

Visit:

http://localhost:8080/capublic

Code

You should see the default Pawtucket front page.

If media thumbnails appear, the symlink is working.

---

## 8. Verify Media Symlink

Inside the container:

docker exec -it ca-container bash
ls -l /var/www/html/capublic/media

Code

You should see:

media -> /var/www/html/ca/media

Code

This ensures Pawtucket uses Providence’s media directory.

---

## 9. Verify URL Routing

Providence must use:

CA_URL_ROOT = "/ca"

Code

Pawtucket must use:

CA_URL_ROOT = "/capublic"

Code

These overrides prevent redirect loops and routing errors.

---

## 10. MySQL Performance Verification

To confirm the tuned buffer pool size:

docker exec -it ca-mysql bash
mysql -u root -prootpass -e "SHOW VARIABLES LIKE 'innodb_buffer_pool_size';"

Code

Expected:

1073741824

Code

This value is applied via the `command:` directive in `docker-compose.yml`.

---

## 11. Installation Complete

You now have:

- Providence running under `/ca`  
- Pawtucket running under `/capublic`  
- MySQL 8.x with a tuned 1GB buffer pool  
- Media shared via symlink  
- PHP 8.x with CA‑specific overrides  
- Apache rewrite enabled  
- A reproducible, cross‑platform CA environment  

Proceed to the configuration and routing guides for next step
