markdown
# Installation Guide

This document explains how to install CollectiveAccess (Providence + Pawtucket) using the Docker Starter Kit. It covers container setup, running the installer, creating the admin account, and verifying the environment.

---

## Prerequisites

Before you begin, ensure you have:

- Docker (latest version)
- Docker Compose
- Git (optional but recommended)
- A web browser

No PHP, MySQL, or Apache installation is required on the host system.

---

## 1. Clone the Starter Kit

Clone the repository or download it as a ZIP:

git clone https://github.com/BasilTas/collectiveaccess-docker-starter-kit.git (github.com in Bing)
cd collectiveaccess-docker-starter-kit/docker

Code

The `docker` folder contains:

- `Dockerfile`
- `docker-compose.yml`
- `php.ini`

These build the CA environment.

---

## 2. Start the Containers

Run:

docker-compose up -d

Code

This launches:

- `ca-app` (Apache + PHP 8.4 + Providence + Pawtucket)
- `ca-mysql` (MySQL 8.x)

You can verify they are running:

docker ps

Code

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
collectiveaccess

Code

**Username:**
root

Code

**Password:**
root

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

When finished, it will display your login credentials.

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

docker exec -it ca-app bash
ls -l /var/www/html/capublic/media

Code

You should see:

media -> /var/www/html/ca/media

Code

If not, recreate the symlink (see `media-symlink.md`).

---

## 9. Verify URL Routing

Providence must use:

CA_URL_ROOT = "/ca"

Code

Pawtucket must use:

CA_URL_ROOT = "/capublic"

Code

These overrides prevent redirect loops and routing errors.

See `routing.md` for details.

---

## 10. Installation Complete

You now have:

- Providence running under `/ca`
- Pawtucket running under `/capublic`
- MySQL connected
- Media shared via symlink
- PHP 8.4 fully supported
- Apache rewrite enabled
- A reproducible CA environment

Proceed to the configuration guide for next steps.
