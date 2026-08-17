# Dockerized WordPress

A reproducible WordPress setup running alongside MariaDB, orchestrated through Docker Compose.

## Table of Contents

- [Quickstart](#quickstart)
- [Usage](#usage)
- [Additional Files](#additional-files)

## Quickstart

### Prerequisites

- Docker
- Docker Compose

### Steps

Clone the repository and move into the project folder:

```bash
git clone git@github.com:Gerth123/dockerized-wordpress.git
cd dockerized-wordpress
```

Copy the environment template and fill in the required values, including the database and root passwords:

```bash
cp .env.example .env
```

Start the setup:

```bash
docker compose up -d
```

Open `http://<your-vm-ip>:8080` in your browser and complete the WordPress setup wizard to create your admin account.

## Usage

The setup consists of two services defined in `docker-compose.yaml`:

| Service | Image | Purpose |
|---|---|---|
| `wordpress` | `wordpress:7.0-apache` | Serves the WordPress application on port `8080` |
| `db` | `mariadb:10.6` | Stores WordPress content and configuration |

Both services use the official Docker Hub images rather than the Bitnami images. Bitnami's versioned tags moved behind a paid subscription in September 2025, leaving only the `:latest` tag free. Relying on `:latest` in a Docker Compose setup means losing the ability to pin a specific version, which makes it impossible to guarantee that the same setup behaves identically across environments or after a rebuild. The official images keep every version freely available, so `wordpress:7.0-apache` here always resolves to the exact same image, regardless of when or where it is pulled.

A few other decisions behind this setup:

**Why official images.** Docker Official Images are reviewed and maintained as part of Docker's official program, not by a random third party. That means the build process and how quickly security fixes get applied are traceable, which is not something you can rely on with every community image.

**Why `wordpress:7.0-apache` instead of a more exact version.** This tag pins the WordPress version to 7.0.x, but not to one exact patch release like 7.0.4. That is a trade off: pinning the exact patch version is more predictable, but means manually updating the tag every time a security patch comes out. Pinning only the minor version still protects against unexpected major changes, while still picking up patch level security fixes when the image is rebuilt.

**Why the Apache variant.** The `-apache` image bundles PHP and the web server together, so the whole app only needs one container. An alternative would be to split PHP (via FPM) and the web server into two separate containers. That is a cleaner separation of concerns, but adds complexity that was not needed for this project.

**Why credentials only live in `.env`.** All passwords are only ever passed in through environment variables from a local, git ignored `.env` file. Neither `docker-compose.yaml` nor `.env.example` ever contain an actual password, only a reference to where the value comes from.

Both services are configured through variables listed in `.env.example`. Copy the template to `.env` before starting the setup:

```bash
cp .env.example .env
```

Non critical values can be set according to your needs:

| Variable | Description |
|---|---|
| `WORDPRESS_DB_HOST` | Hostname of the database service |
| `WORDPRESS_DB_NAME` | Name of the WordPress database |
| `WORDPRESS_DB_USER` | Database username |

Credential values must also be set, but **must not be committed to the repository**. They only exist in your local, git-ignored `.env` file:

| Variable | Description |
|---|---|
| `WORDPRESS_DB_PASSWORD` | Password for the database user |
| `MARIADB_ROOT_PASSWORD` | Root password for the MariaDB instance |

Docker Compose automatically picks up values from `.env` and uses them in place of the placeholders in `docker-compose.yaml`, without you needing to touch the tracked file.

WordPress is reachable on port `8080` of the host. To use a different port, adjust the port mapping for the `wordpress` service in `docker-compose.yaml`.

Unlike the Bitnami image, the official WordPress image does not create an admin account through environment variables. On first visiting `http://<your-vm-ip>:8080`, WordPress runs its own five minute setup wizard, where you choose the site title and set the admin username and password directly in the browser. This keeps the image close to the upstream WordPress distribution, without additional automation scripts layered on top.

The site files and the database are stored in the named Docker volumes `wordpress_data` and `db_data`. This means content, plugins, and database entries survive container restarts and rebuilds. To reset the installation entirely, remove the volumes before starting again:

```bash
docker compose down -v
```

Both services are configured with a `restart: unless-stopped` policy. If a container terminates unexpectedly due to an error, Docker will automatically restart it.

## Additional Files

- `docker-compose.yaml`: defines the `wordpress` and `db` services, their volumes, and environment configuration.
- `.env.example`: a template showing which variables can be set in a local `.env` file, including which ones require credentials. Copy it to `.env` and adjust values there; `.env` itself is git-ignored.
- `docs/Wordpress Checkliste.pdf`: the official project checklist provided by Developer Akademie, kept here for reference during development. It is not part of the application itself.