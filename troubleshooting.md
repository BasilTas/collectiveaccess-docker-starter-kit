# Troubleshooting Guide

This document covers common issues encountered when running CollectiveAccess inside the Docker Starter Kit. It includes fixes for MySQL errors, routing problems, media issues, installation failures, and Docker‑specific behaviour on Windows, macOS, and Linux.

---

# 1. MySQL Problems

## 1.1 MySQL fails to start with `binlog.index` errors

### Symptoms
- MySQL container exits immediately  
- Logs show:
binlog.index: permission denied

Code
- Providence reports “database not installed”

### Cause
MySQL data directories are **not portable** between distributions.  
Switching images (Debian ↔ Alpine) breaks the existing volume.

### Fix
Delete the MySQL data volume:

docker volume rm collectiveaccess-docker_mysql_data
docker-compose up -d

Code

This is normal and unavoidable.

---

## 1.2 MySQL ignores custom config files

### Symptoms
- `innodb_buffer_pool_size` stays at 134217728  
- No warnings in logs  
- Mounted config file appears in container but is not applied

### Cause
On Windows, MySQL config mounts may appear but **not be readable** internally.  
Some MySQL images silently ignore mounted config files.

### Fix
Use the `command:` override:

```yaml
command: --innodb-buffer-pool-size=1G
This method is reliable across all platforms.

1.3 MySQL buffer pool size is incorrect
Check
Code
docker exec -it ca-mysql bash
mysql -u root -prootpass -e "SHOW VARIABLES LIKE 'innodb_buffer_pool_size';"
Expected:

Code
1073741824
If not:

Ensure the command: override is present

Recreate the container

Delete the MySQL volume if switching images

2. Docker & YAML Problems
2.1 additional properties 'mysql' not allowed
Cause
Indentation error in docker-compose.yml:

Code
services:
  ca:
    ...
 mysql:   ← incorrect indentation
Fix
Ensure mysql: is aligned under services::

yaml
services:
  ca:
    ...
  mysql:
    ...
2.2 mapping key "volumes" already defined
Cause
Duplicate volumes: block or mis‑indentation.

Fix
Ensure only one top‑level volumes: block exists:

yaml
volumes:
  mysql_data:
3. Installation Problems
3.1 Installer takes 800+ seconds
Cause
Docker Desktop on Windows has slow filesystem performance.
CA installer is CPU‑bound and reads thousands of files.

Fix
This is normal.

Typical install times:

Windows + Docker Desktop: ~800 seconds

WSL2: ~300–400 seconds

Native Linux: ~200–300 seconds

Runtime performance improves dramatically after installation.

3.2 Installer reports “database not installed”
Causes
MySQL volume reset

MySQL failed to start

Wrong database host

Fixes
Ensure MySQL is running

Use hostname:

Code
mysql
Re-run installer:

Code
http://localhost:8080/ca/install
4. Routing Problems
4.1 Redirect loop when accessing Providence
Cause
Incorrect URL root.

Fix
In ca/app/conf/local/configuration.php:

php
__CA_URL_ROOT__ = "/ca";
4.2 Pawtucket loads Providence pages
Cause
Incorrect URL root or missing Apache alias.

Fix
In capublic/app/conf/local/configuration.php:

php
__CA_URL_ROOT__ = "/capublic";
Ensure Apache alias exists:

apache
Alias /capublic /var/www/html/capublic
4.3 “Not Found” or missing CSS/JS
Cause
Apache rewrite disabled.

Fix
Ensure:

apache
AllowOverride All
is set for both /ca and /capublic.

5. Media Problems
5.1 Pawtucket thumbnails broken
Cause
Missing or incorrect symlink.

Fix
Inside container:

Code
rm -rf /var/www/html/capublic/media
ln -s /var/www/html/ca/media /var/www/html/capublic/media
5.2 Providence derivatives missing
Cause
Permissions incorrect.

Fix
Code
chmod -R 775 ca/media
chown -R www-data:www-data ca/media
6. Configuration Problems
6.1 Wrong database host
Fix
Always use:

Code
mysql
Never localhost, 127.0.0.1, or container IPs.

6.2 Wrong URL root
Fix
Providence:

php
__CA_URL_ROOT__ = "/ca";
Pawtucket:

php
__CA_URL_ROOT__ = "/capublic";
7. Docker Problems
7.1 Containers start but CA shows blank page
Causes
PHP error

missing extensions

incorrect volume mount

Fix
Check logs:

Code
docker logs ca-app
7.2 MySQL container starts but CA cannot connect
Causes
MySQL still initializing

wrong credentials

wrong hostname

Fix
Wait 10–20 seconds after startup.
Verify credentials in configuration.php.

8. Windows‑Specific Issues
8.1 MySQL config mounts ignored
Fix: use command: override.

8.2 Installer slow
Fix: normal behaviour under Docker Desktop.

8.3 Bind‑mount appears but is unreadable
Fix: avoid MySQL config mounts entirely.

9. Useful Debugging Commands
Enter CA container
Code
docker exec -it ca-app bash
Enter MySQL container
Code
docker exec -it ca-mysql bash
Check MySQL logs
Code
docker logs ca-mysql
Check Apache/PHP logs
Code
docker logs ca-app
Restart containers
Code
docker-compose restart
Rebuild environment
Code
docker-compose down
docker-compose up -d --force-recreate
10. Summary
This troubleshooting guide covers:

MySQL startup failures

buffer pool tuning issues

config‑mount problems

routing errors

media symlink issues

installer behaviour

Docker Desktop quirks

YAML indentation traps

Use this document to diagnose and resolve issues quickly when working with the CollectiveAccess Docker Starter Kit.
