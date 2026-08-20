# Configuration Guide

This document explains how to configure the CollectiveAccess Docker Starter Kit, including environment variables, PHP settings, Apache routing, MySQL tuning, and application‑level configuration for Providence and Pawtucket.

The goal is to provide a predictable, reproducible configuration workflow that works reliably across Windows, macOS, and Linux.

---

## 1. Environment Variables

Environment variables for MySQL are defined in `docker-compose.yml` under the `mysql` service:

```yaml
environment:
  MYSQL_ROOT_PASSWORD: rootpass
  MYSQL_DATABASE: ca
  MYSQL_USER: causer
  MYSQL_PASSWORD: capass
These values are used by the installer and by Providence/Pawtucket when connecting to the database.

If you change these values, update the CA configuration files accordingly.

2. Database Connection Configuration
After installation, CollectiveAccess stores database connection settings in:

Code
ca/app/conf/local/configuration.php
capublic/app/conf/local/configuration.php
The installer writes these automatically.
If you need to adjust them manually, the relevant entries are:

php
__CA_DB_HOST__ = "mysql";
__CA_DB_USER__ = "causer";
__CA_DB_PASSWORD__ = "capass";
__CA_DB_DATABASE__ = "ca";
The hostname must remain mysql, which is the internal Docker network name of the database container.

3. MySQL Configuration
MySQL Image
The starter kit uses:

Code
mysql/mysql-server:8.0
This image is stable, predictable, and works consistently across platforms.

Buffer Pool Tuning
The InnoDB buffer pool size is set using the container command: directive:

yaml
command: --innodb-buffer-pool-size=1G
This method is used because:

MySQL config file mounts are unreliable on Windows

Alpine and Debian MySQL images use different config directories

The buffer pool is critical for CollectiveAccess performance

The command: override works consistently everywhere

Data Persistence
MySQL stores its data in a Docker volume:

Code
mysql_data
If you switch MySQL images, you must delete this volume:

Code
docker volume rm collectiveaccess-docker_mysql_data
This is required because MySQL data directories are not portable between distributions.

4. PHP Configuration
The starter kit includes a custom php.ini file mounted into the container:

Code
./php.ini:/usr/local/etc/php/php.ini
This file applies performance‑critical overrides for CollectiveAccess.

Recommended Settings
The following settings are included and should remain:

ini
memory_limit = 1024M
upload_max_filesize = 512M
post_max_size = 512M
max_execution_time = 300
max_input_time = 300
These values ensure:

large media uploads work

long‑running imports do not time out

CA’s installer and metadata builder complete reliably

If you need to adjust PHP behaviour, modify php.ini and restart the containers.

5. Apache Configuration
Apache configuration is provided by:

Code
./apache.conf:/etc/apache2/sites-enabled/000-default.conf
This file defines:

document root

routing for /ca and /capublic

rewrite rules

directory permissions

PHP handler behaviour

The default configuration supports both Providence and Pawtucket without requiring virtual hosts.

6. Application Configuration (Providence & Pawtucket)
URL Root Overrides
The installer writes URL settings into:

Code
ca/app/conf/local/configuration.php
capublic/app/conf/local/configuration.php
The starter kit ensures correct routing by setting:

php
__CA_URL_ROOT__ = "/ca";
__CA_URL_ROOT__ = "/capublic";
These values match the Apache routing and Docker port mapping.

Media Directory
Pawtucket uses a symlink:

Code
capublic/media → ca/media
This ensures:

no duplication

consistent derivatives

correct public access

correct backend access

No configuration changes are required for media handling.

7. Resetting the Environment
If you need to reset the stack:

Stop containers

Code
docker-compose down
Remove MySQL data volume

Code
docker volume rm collectiveaccess-docker_mysql_data
Rebuild

Code
docker-compose up -d --force-recreate
Re-run the installer

Code
http://localhost:8080/ca/install
8. Known Windows Considerations
When running under Docker Desktop on Windows:

MySQL config file mounts may appear but not be readable

The installer may take ~800 seconds due to filesystem overhead

PHP file scanning is slower than on native Linux

The buffer pool must be set via command: rather than config files

These behaviours are normal and not signs of misconfiguration.

Summary
This configuration approach provides:

reliable MySQL tuning

predictable PHP behaviour

correct routing for both CA applications

stable media handling

reproducible Docker builds

cross‑platform compatibility

Use this document as the reference for configuring and maintaining your CollectiveAccess Docker environment.
