# Media Directory Symlink

This document explains how media files are shared between Providence and Pawtucket in the CollectiveAccess Docker Starter Kit. It covers why the symlink is required, how to create it, how to verify it, and how CA uses the shared directory internally.

---

## Overview

CollectiveAccess stores all media files (images, audio, video, documents) in the **Providence** installation. Pawtucket does not maintain its own media directory.

To allow Pawtucket to access Providence’s media, a **symbolic link** is used:

```text
capublic/
└── media → ../ca/media
```

Code

This ensures:

- no duplication of media files  
- consistent derivative generation  
- correct public access  
- correct backend access  
- correct thumbnail and viewer behaviour  

This symlink mirrors the recommended CA deployment pattern used on Linux, macOS, and Windows (via Docker).

---

## Why Pawtucket Needs a Symlink

### 1. Providence generates all derivatives  
Thumbnails, previews, zoomable images, and other derivatives are created by Providence and stored under:

ca/media/

Code

### 2. Pawtucket expects the same directory structure  
Pawtucket’s templates and media viewers assume:

capublic/media/

Code

### 3. Pawtucket must not duplicate media  
If Pawtucket had its own media directory:

- derivatives would be missing  
- uploads would be inconsistent  
- storage would double  
- media viewers would break  

### 4. CA’s internal media loader follows symlinks  
The CA media loader resolves symlinks correctly, making this approach fully supported.

---

## Creating the Symlink (inside Docker)

Enter the CA application container:

docker exec -it ca-app bash

Code

Remove Pawtucket’s default media directory:

rm -rf /var/www/html/capublic/media

Code

Create the symbolic link:

ln -s /var/www/html/ca/media /var/www/html/capublic/media

Code

---

## Verifying the Symlink

Run:

ls -l /var/www/html/capublic/media

Code

Expected output:

media -> /var/www/html/ca/media

Code

If you see this, the symlink is correct.

---

## Apache Requirements

Most Apache configurations already allow symlinks.  
If needed, ensure:

Options FollowSymLinks

Code

is enabled for the Pawtucket directory.

This is rarely required in Docker, but important in some custom deployments.

---

## How CA Uses the Symlink Internally

### Providence
- Stores original media  
- Generates derivatives  
- Stores derivatives under `ca/media/`  
- Serves media to Pawtucket through the symlink  

### Pawtucket
- Reads media through `capublic/media/`  
- Uses Providence’s derivatives  
- Displays thumbnails, viewers, and zoomable images  
- Never writes to the media directory  

This ensures consistent behaviour across both applications.

---

## Common Issues

### Pawtucket shows broken thumbnails
Cause:
- symlink missing or incorrect

Fix:
- recreate symlink

### Pawtucket shows “Media not found”
Cause:
- incorrect `__CA_URL_ROOT__`
- incorrect routing

Fix:
- verify `/capublic` routing (see `routing.md`)

### Providence derivatives missing
Cause:
- permissions incorrect

Fix:
chmod -R 775 /var/www/html/ca/media
chown -R www-data:www-data /var/www/html/ca/media

Code

---

## Windows Notes

When running under Docker Desktop on Windows:

- The media symlink works normally  
- Windows bind‑mount quirks do **not** affect the symlink  
- All media operations occur inside the Linux container filesystem  

This makes media handling one of the most stable parts of the CA stack.

---

## Summary

The media symlink is essential for correct CA behaviour:

- Providence stores and generates all media  
- Pawtucket reads media through a symlink  
- No duplication  
- No broken thumbnails  
- No missing derivatives  
- Fully supported by CA’s media loader  

See `routing.md` for details on URL handling and `troubleshooting.md` for common media-related issues.
