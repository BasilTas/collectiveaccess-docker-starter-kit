# CollectiveAccess Docker Starter Kit
[Note: since working through this process, it has become apparent that even with steps taken to maximise resources, CollectiveAccess in a Docker environment, even on an up-to-date Windows 11 PC, is painfully slow in operation, and slower than running under Windows IIS. Further investigations have proved that the fastest option in a Windows environment is to set it up entirely within a WSL2 (Windows Subsystem for Linux) environment. This is substantially faster than an existing Windows IIS instance we have been using for several years.] See See: https://github.com/BasilTas/collectiveaccess-for-windows-wsl

A modern, reproducible Docker environment for running **CollectiveAccess** (Providence + Pawtucket) with:

- PHP 8.x  
- Apache  
- MySQL 8.x (tuned)  
- Correct routing  
- Shared media directory  
- Stable cross‑platform behaviour  
- Clean, predictable configuration  

This starter kit is designed for museums, archives, libraries, researchers, and developers who want a reliable CA environment without the complexity of manual installation.

---

## Features

- Providence and Pawtucket served from a single Apache/PHP container  
- MySQL 8.x with a **1GB InnoDB buffer pool** for fast metadata and search performance  
- Correct URL routing (`/ca` and `/capublic`)  
- Shared media directory via symlink  
- Clean directory layout matching CA best practices  
- Cross‑platform support (Windows, macOS, Linux)  
- Fully documented architecture and configuration  
- Reproducible builds using Docker Compose  

---

## Quick Start

Clone the repository:

git clone https://github.com/BasilTas/collectiveaccess-docker-starter-kit
cd collectiveaccess-docker-starter-kit

Code

Start the environment:

docker-compose up -d

Code

Run the Providence installer:

http://localhost:8080/ca/install

Code

Use these database settings:

Host: mysql
Database: ca
User: causer
Password: capass

Code

After installation:

- Providence → `http://localhost:8080/ca`  
- Pawtucket → `http://localhost:8080/capublic`  

---

## Directory Layout
```text
collectiveaccess-docker/
│
├── ca/               ← Providence
│   ├── app/
│   ├── media/
│   ├── themes/
│   ├── vendor/
│   ├── setup.php
│   └── post-setup.php
│
├── capublic/         ← Pawtucket
│   ├── app/
│   ├── media → symlink to ../ca/media
│   ├── themes/
│   ├── vendor/
│   ├── setup.php
│   └── post-setup.php
│
├── php.ini           ← PHP overrides
├── apache.conf       ← Apache routing config
├── docker-compose.yml
└── docs/             ← Full documentation set
```
Code

---

## MySQL Performance Tuning

The MySQL container uses:

mysql/mysql-server:8.0

Code

The InnoDB buffer pool is set via `command:`:

```yaml
command: --innodb-buffer-pool-size=1G
This method is:

reliable on Windows, macOS, and Linux

independent of Alpine/Debian differences

guaranteed to apply

essential for CA performance

Verify the buffer pool:

Code
docker exec -it ca-mysql bash
mysql -u root -prootpass -e "SHOW VARIABLES LIKE 'innodb_buffer_pool_size';"
Expected:

Code
1073741824
Routing
Providence:

Code
http://localhost:8080/ca
Pawtucket:

Code
http://localhost:8080/capublic
URL roots are set automatically by the installer:

Providence:

php
__CA_URL_ROOT__ = "/ca";
Pawtucket:

php
__CA_URL_ROOT__ = "/capublic";
Media Handling
Pawtucket uses a symlink:

Code
capublic/media → ca/media
This ensures:

no duplication

correct thumbnails

correct derivatives

correct viewer behaviour

Documentation
Full documentation is available in the /docs directory:

architecture.md — container layout & directory structure

installation.md — step‑by‑step installation

configuration.md — PHP, MySQL, Apache, CA settings

routing.md — URL roots, rewrite rules, aliases

media-symlink.md — shared media directory

migrating.md — migrating existing CA installations

upgrading.md — safe upgrade workflow

troubleshooting.md — common issues & fixes

faq.md — quick answers

Resetting the Environment
If you need to reset MySQL:

Code
docker-compose down
docker volume rm collectiveaccess-docker_mysql_data
docker-compose up -d
Then re-run the installer.

Notes for Windows Users
When running under Docker Desktop:

CA installer may take ~800 seconds (normal)

MySQL config mounts may be ignored

Buffer pool must be set via command:

Media symlink works normally

Runtime performance is excellent once installed

License
This starter kit is provided as-is for educational and institutional use.
See CollectiveAccess licensing for application-specific terms.

Contributing
Pull requests are welcome.
If you have improvements, documentation updates, or fixes, feel free to submit them.
