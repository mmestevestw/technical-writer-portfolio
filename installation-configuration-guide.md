# Snipe-IT Installation and Configuration Guide (Docker) — Sample

> **Note:** This is a writing sample demonstrating install/configuration documentation style. It walks through a real, working Docker-based setup of the open-source [Snipe-IT](https://github.com/grokability/snipe-it) asset management platform.

## 1. Overview

This guide covers deploying Snipe-IT using Docker Compose, a supported and commonly used deployment method for self-hosted instances. It's intended for an IT administrator setting up a new instance, either for evaluation or production use.

**What you'll end up with:** a running Snipe-IT web application, backed by a MySQL database, accessible over HTTP(S), with an initial admin account configured.

## 2. Prerequisites

Before starting, confirm the following:

| Requirement | Notes |
|---|---|
| Docker Engine | Version 20.10 or later. |
| Docker Compose | v2 (the `docker compose` subcommand, not the standalone `docker-compose` binary). |
| Available disk space | Minimum 2 GB for images and volumes; more depending on attached files (asset photos, license files). |
| Open port | 80 or 443 available on the host, or a reverse proxy already routing to the container. |
| Outbound network access | Required during image pull and for any email/SMTP integration. |

**Do:** confirm your host clock is synchronized (NTP). Snipe-IT's license and depreciation calculations are date-sensitive, and clock drift can cause inconsistent reporting.

## 3. Step-by-Step Installation

### 3.1 Create the project directory

```bash
mkdir snipeit-deploy && cd snipeit-deploy
```

### 3.2 Create the Docker Compose file

Create `docker-compose.yml`:

```yaml
version: "3.8"

services:
  snipeit:
    image: snipe/snipe-it:latest
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      - APP_URL=http://localhost:8080
      - APP_KEY=
      - APP_TIMEZONE=America/Sao_Paulo
      - DB_CONNECTION=mysql
      - DB_HOST=snipeit-db
      - DB_DATABASE=snipeit
      - DB_USERNAME=snipeit
      - DB_PASSWORD=change-this-password
    volumes:
      - snipeit-data:/var/lib/snipeit
    depends_on:
      - snipeit-db

  snipeit-db:
    image: mysql:8.0
    restart: unless-stopped
    environment:
      - MYSQL_ROOT_PASSWORD=change-this-root-password
      - MYSQL_DATABASE=snipeit
      - MYSQL_USER=snipeit
      - MYSQL_PASSWORD=change-this-password
    volumes:
      - snipeit-db-data:/var/lib/mysql

volumes:
  snipeit-data:
  snipeit-db-data:
```

**Stop:** do not leave `change-this-password` and `change-this-root-password` as literal values outside of local evaluation environments. Replace them with strong, unique credentials, and consider using Docker secrets or an environment file excluded from version control for production deployments.

### 3.3 Generate an application key

Snipe-IT requires a unique `APP_KEY` for encryption. Generate one after the containers are running:

```bash
docker compose up -d
docker compose exec snipeit php artisan key:generate --show
```

Copy the output value (it will look like `base64:XXXXXXXX...`) into the `APP_KEY` environment variable in `docker-compose.yml`, then restart:

```bash
docker compose up -d --force-recreate snipeit
```

### 3.4 Verify the containers are healthy

```bash
docker compose ps
```

Both `snipeit` and `snipeit-db` should show a state of `running` (or `healthy`, if healthchecks are configured). If `snipeit` is restarting repeatedly, check logs before continuing:

```bash
docker compose logs snipeit --tail=100
```

### 3.5 Complete the setup wizard

1. Open `http://localhost:8080` in a browser.
2. The setup wizard checks PHP extensions and file permissions automatically; resolve any items flagged in red before proceeding.
3. Confirm the database connection settings (pre-filled from your environment variables).
4. Create the initial admin user.

## 4. Post-Installation Configuration

### 4.1 Configure email (optional but recommended)

Set the following environment variables to enable password resets and notifications:

```yaml
- MAIL_MAILER=smtp
- MAIL_HOST=smtp.yourprovider.com
- MAIL_PORT=587
- MAIL_USERNAME=notifications@yourcompany.com
- MAIL_PASSWORD=your-smtp-password
- MAIL_ENCRYPTION=tls
```

### 4.2 Set up scheduled tasks

Snipe-IT relies on Laravel's task scheduler for maintenance jobs (license expiration checks, report generation). Add a cron entry to trigger it every minute:

```bash
* * * * * docker compose exec -T snipeit php artisan schedule:run >> /dev/null 2>&1
```

### 4.3 Enable HTTPS

For any deployment beyond local evaluation, place a reverse proxy (nginx, Traefik, or Caddy) in front of the container to terminate TLS, and update `APP_URL` to reflect the `https://` address. Snipe-IT will refuse to set secure cookies correctly if `APP_URL` doesn't match the scheme users actually connect with.

## 5. Troubleshooting

| Symptom | Likely Cause | Resolution |
|---|---|---|
| Container restarts in a loop | `APP_KEY` missing or invalid | Regenerate the key per §3.3 and confirm it's present in the environment before the container starts. |
| "500 Server Error" on first load | File permission issue on the `storage` volume | Run `docker compose exec snipeit chown -R www-data:www-data /var/lib/snipeit/storage`. |
| Setup wizard can't reach the database | `DB_HOST` doesn't match the Compose service name | Confirm `DB_HOST` is set to the database service's name in `docker-compose.yml` (`snipeit-db` in this example), not `localhost`. |
| Emails not sending | SMTP credentials incorrect, or port blocked outbound | Test SMTP credentials independently (e.g., with `swaks`) before troubleshooting further inside the container. |

## 6. Backup Recommendations

At minimum, back up:

- The `snipeit-db-data` volume (database).
- The `snipeit-data` volume (uploaded files: asset photos, license attachments).
- Your `docker-compose.yml` and any external environment file.

A simple approach for the database:

```bash
docker compose exec snipeit-db mysqldump -u root -p snipeit > snipeit-backup-$(date +%F).sql
```

---
*Sample documentation written by Matheus [Last Name] as a portfolio exercise, based on a real, working Docker deployment of Snipe-IT.*
