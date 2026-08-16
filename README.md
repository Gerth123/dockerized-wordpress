# Dockerized WordPress

A reproducible WordPress setup running alongside MariaDB, orchestrated through Docker Compose.

## Table of Contents

- [Quickstart](#quickstart)
- [Project Goal](#project-goal)
- [Usage](#usage)
- [Additional Files](#additional-files)

## Quickstart

### Prerequisites

- Docker
- Docker Compose

### Steps

Clone the repository and move into the project folder:

```bash
git clone https://github.com/Gerth123/dockerized-wordpress.git
cd dockerized-wordpress
```

Copy the environment template and fill in the required values, including the database and admin passwords:

```bash
cp .env.example .env
```

Start the setup:

```bash
docker compose up -d
```

Once it's running, open `http://<your-vm-ip>:8080` in your browser and log in with the admin credentials configured in `.env`.

## Project Goal

This repository provides a Dockerized WordPress setup running alongside a MariaDB database, orchestrated through Docker Compose. The `docker-compose.yaml` runs two services, `wordpress` and `db`, exposes WordPress on port 8080, and persists both the site files and the database in Docker volumes so nothing is lost on restart. Configuration happens entirely through environment variables, keeping credentials and other sensitive values out of the codebase.

## Usage

The setup consists of two services defined in `docker-compose.yaml`:

| Service | Image | Purpose |
|---|---|---|
| `wordpress` | `bitnami/wordpress:latest` | Serves the WordPress application on port `8080` |
| `db` | `mariadb:10.6` | Stores WordPress content and configuration |

Both services are configured through variables listed in `.env.example`. Copy the template to `.env` before starting the setup:

```bash
cp .env.example .env
```

Non critical values can be set according to your needs:

| Variable | Description |
|---|---|
| `WORDPRESS_DATABASE_HOST` | Hostname of the database service |
| `WORDPRESS_DATABASE_NAME` | Name of the WordPress database |
| `WORDPRESS_DATABASE_USER` | Database username |
| `WORDPRESS_USERNAME` | WordPress admin username |

Credential values must also be set, but **must not be committed to the repository**. They only exist in your local, git-ignored `.env` file:

| Variable | Description |
|---|---|
| `WORDPRESS_DATABASE_PASSWORD` | Password for the database user |
| `WORDPRESS_PASSWORD` | Password for the WordPress admin user |
| `MARIADB_ROOT_PASSWORD` | Root password for the MariaDB instance |

Docker Compose automatically picks up values from `.env` and uses them in place of the placeholders in `docker-compose.yaml`, without you needing to touch the tracked file.

WordPress is reachable on port `8080` of the host. To use a different port, adjust the port mapping for the `wordpress` service in `docker-compose.yaml`.

The site files and the database are stored in the named Docker volumes `wordpress_data` and `db_data`. This means content, plugins, and database entries survive container restarts and rebuilds. To reset the installation entirely, remove the volumes before starting again:

```bash
docker compose down -v
```

Both services are configured with a `restart: unless-stopped` policy. If a container terminates unexpectedly due to an error, Docker will automatically restart it.

## Additional Files

- `docker-compose.yaml`: defines the `wordpress` and `db` services, their volumes, and environment configuration.
- `.env.example`: a template showing which variables can be set in a local `.env` file, including which ones require credentials. Copy it to `.env` and adjust values there; `.env` itself is git-ignored.
- `docs/Wordpress Checkliste.pdf`: the official project checklist provided by Developer Akademie, kept here for reference during development. It is not part of the application itself.