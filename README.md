# collectiveaccess-docker-starter-kit
Modern Docker setup for CollectiveAccess (Providence + Pawtucket) with PHP 8.4, MySQL, correct routing, symlinked media, and full documentation.

CollectiveAccess Docker Starter Kit
A modern, clean Docker setup for CollectiveAccess Providence and Pawtucket, designed for museums, archives, libraries, and researchers who want a reliable, reproducible CA environment without the pain of manual installation.

This starter kit provides:

PHP 8.4 (fully compatible with current CA releases)

MySQL 8.x

Apache with rewrite enabled

Providence served at /ca

Pawtucket served at /capublic

Shared media directory via Linux symlink

Correct URL routing (no redirect loops)

Clean, documented configuration files

A simple, reproducible Docker workflow

This repository focuses on documentation and configuration, not distributing a prebuilt container image. You build your own CA environment using the included files.

Quick Start
1. Clone this repository
Code
git clone https://github.com/BasilTas/collectiveaccess-docker-starter-kit.git
cd collectiveaccess-docker-starter-kit/docker
2. Start the containers
Code
docker-compose up -d
This launches:

ca-app (Apache + PHP 8.4 + Providence + Pawtucket)

ca-mysql (MySQL 8.x)

3. Install Providence
Open your browser:

Code
http://localhost:8080/ca/install
Run the installer, create your admin account, and complete setup.

4. Access Providence
Code
http://localhost:8080/ca
5. Access Pawtucket
Code
http://localhost:8080/capublic
Included Files
docker/Dockerfile
Builds a modern PHP 8.4 + Apache environment with all required CA extensions.

docker/docker-compose.yml
Defines the CA application container and MySQL database container.

docker/php.ini
Provides recommended PHP settings for CA (upload limits, memory, timezone, etc).

Important Fixes Included
This starter kit includes solutions for several long‑standing CA deployment issues:

Correct __CA_URL_ROOT__ for Providence (/ca)

Correct __CA_URL_ROOT__ for Pawtucket (/capublic)

Symlinked media directory (capublic/media → ca/media)

Correct MySQL hostname (mysql) inside Docker

Correct Apache rewrite rules

Correct permissions for CA’s app/tmp directories

Fix for infinite login redirect loops

Correct installer URL routing

These fixes are documented in detail in the /docs folder.

Full Documentation
The full professional documentation set is available in:

Code
docs/
It includes:

Architecture overview

Installation guide

Configuration guide

Routing and URL root fixes

Media symlink explanation

Troubleshooting

Upgrading CA

FAQ

This documentation is designed to help both beginners and institutions deploying CA in production.

Contributing
If you find improvements or want to add examples, feel free to open an issue or submit a pull request.
This starter kit is intended to help the CollectiveAccess community modernise and simplify deployments.

License
This project is provided as‑is for educational and deployment use.
CollectiveAccess itself is licensed under the GNU Public License (GPL).
