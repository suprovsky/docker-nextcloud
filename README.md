# suprovsky/nextcloud

## About

This non-official image is intended as an **all-in-one** (as in monolithic) Nextcloud **production** image based on [wonderfall/docker-nextcloud](https://github.com/wonderfall/docker-nextcloud) with an inclusion of imagick plugin.
___

- [suprovsky/nextcloud](#suprovskynextcloud)
  - [About](#about)
  - [Features](#features)
  - [Security](#security)
  - [Tags](#tags)
  - [Build-time variables](#build-time-variables)
  - [Environment variables](#environment-variables)
    - [Runtime](#runtime)
    - [Startup](#startup)
  - [Volumes](#volumes)
  - [Ports](#ports)
  - [Migration](#migration)
  - [Usage](#usage)

## Features

- Based on [Alpine Linux](https://alpinelinux.org/).
- Fetching PHP/nginx from their official images.
- **Rootless**: no privilege at any time, even at startup.
- Uses [s6](https://skarnet.org/software/s6/) as a lightweight process supervisor.
- Supports MySQL/MariaDB, PostgresQL and SQLite3 database backends.
- Includes OPcache and APCu for improved caching & performance, also supports redis.
- Tarball integrity & authenticity checked during build process.
- Includes **hardened_malloc**, [a hardened memory allocator](https://github.com/GrapheneOS/hardened_malloc).
- Includes **Snuffleupagus**, [a PHP security module](https://github.com/jvoisin/snuffleupagus).
- Includes a simple **built-in cron** system.
- Much easier to maintain thanks to multi-stages build.
- Includes imagick by default.

You're free to make your own image based on this one if you want a specific feature. Uncommon features won't be included as they can increase attack surface: this image intends to stay **minimal**, but **functional enough** to cover basic needs.

## Security

Don't run random images from random dudes on the Internet. Ideally, you want to maintain and build it yourself.

- **Images are scanned every day** by [Trivy](https://github.com/aquasecurity/trivy) for OS vulnerabilities. Known vulnerabilities will be automatically uploaded to [GitHub Security Lab](https://github.com/suprovsky/docker-nextcloud/security/code-scanning) for full transparency. This also warns me if I have to take action to fix a vulnerability.
- **Latest tag/version is automatically built weekly**, so you should often update your images regardless if you're already using the latest Nextcloud version.
- **Build production images without cache** (use `docker build --no-cache` for instance) if you want to build your images manually. Latest dependencies will hence be used instead of outdated ones due to a cached layer.
- **A security module for PHP called [Snuffleupagus](https://github.com/jvoisin/snuffleupagus) is used by default**. This module aims at killing entire bug and security exploit classes (including XXE, weak PRNG, file-upload based code execution), thus raising the cost of attacks. For now we're using a configuration file derived from [the default one](https://github.com/jvoisin/snuffleupagus/blob/master/config/default_php8.rules), with some explicit exceptions related to Nextcloud. This configuration file is tested and shouldn't break basic functionality, but it can cause issues in specific and untested use cases: if that happens to you, get logs from either `syslog` or `/nginx/logs/error.log` inside the container, and [open an issue](https://github.com/suprovsky/docker-nextcloud/issues). You can also disable the security module altogether by changing the `PHP_HARDENING` environment variable to `false` before recreating the container.
- **Images are signed with the GitHub-provided OIDC token in Actions** using the experimental "keyless" signing feature provided by [cosign](https://github.com/sigstore/cosign). You can verify the image signature using `cosign` as well:

```env
COSIGN_EXPERIMENTAL=true cosign verify ghcr.io/suprovsky/nextcloud
```

Verifying the signature isn't a requirement, and might not be as seamless as using *Docker Content Trust* (which is not supported by GitHub's OCI registry). However, it's strongly recommended to do so in a sensitive environment to ensure the authenticity of the images and further limit the risk of supply chain attacks.

## Tags

- `latest` : latest Nextcloud version
- `x` : latest Nextcloud x.x (e.g. `27`)
- `x.x.x` : Nextcloud x.x.x (e.g. `27.0.0`)

You can always have a glance [here](https://github.com/users/suprovsky/packages/container/package/nextcloud).
Only the **latest stable version** will be maintained by myself.

*Note: automated builds only target `linux/amd64` (x86_64). There is no technical reason preventing the image to be built for `arm64` (in fact you can build it yourself), but GitHub Actions runners are limited in memory, and this limit makes it currently impossible to target both platforms.*

## Build-time variables

|          Variable           |               Description              |       Default      |
| --------------------------- | -------------------------------------- | ------------------ |
| **NEXTCLOUD_VERSION**       | version of Nextcloud                   |          *         |
| **ALPINE_VERSION**          | version of Alpine Linux                |          *         |
| **PHP_VERSION**             | version of PHP                         |          *         |
| **NGINX_VERSION**           | version of nginx                       |          *         |
| **HARDENED_MALLOC_VERSION** | version of hardened_malloc             |          *         |
| **SNUFFLEUPAGUS_VERSION**   | version of Snuffleupagus (php ext)     |          *         |
| **SHA256_SUM**              | checksum of Nextcloud tarball (sha256) |          *         |
| **GPG_FINGERPRINT**         | fingerprint of Nextcloud GPG key       |          *         |
| **UID**                     | user id                                |        1000        |
| **GID**                     | group id                               |        1000        |
| **CONFIG_NATIVE**           | native code for hardened_malloc        |        false       |
| **VARIANT**                 | variant of hardened_malloc (see repo)  |        light       |

(latest known available, likely to change regularly)

For convenience they were put at [the very top of the Dockerfile](https://github.com/suprovsky/docker-nextcloud/blob/main/Dockerfile#L1-L13) and their usage should be quite explicit if you intend to build this image yourself. If you intend to change `NEXTCLOUD_VERSION`, change `SHA256_SUM` accordingly.

## Environment variables

### Runtime

|          Variable         |         Description         |       Default      |
| ------------------------- | --------------------------- | ------------------ |
|     **UPLOAD_MAX_SIZE**   | file upload maximum size    |         10G        |
|      **APC_SHM_SIZE**     | apc shared memory size      |         128M       |
|    **OPCACHE_MEM_SIZE**   | opcache available memory    |         128M       |
|      **MEMORY_LIMIT**     | max php command mem usage   |         512M       |
|       **CRON_PERIOD**     | cron time interval (min.)   |         5m         |
|   **CRON_MEMORY_LIMIT**   | cron max memory usage       |         1G         |
|         **DB_TYPE**       | sqlite3, mysql, pgsql       |       sqlite3      |
|         **DOMAIN**        | host domain                 |      localhost     |
|      **PHP_HARDENING**    | enables snuffleupagus       |        true        |

Leave them at default if you're not sure what you're doing.

### Startup

|          Variable         |         Description         |
| ------------------------- | --------------------------- |
|        **ADMIN_USER**     | admin username              |
|      **ADMIN_PASSWORD**   | admin password              |
|         **DB_TYPE**       | sqlite3, mysql, pgsql       |
|         **DB_NAME**       | name of the database        |
|         **DB_USER**       | name of the database user   |
|       **DB_PASSWORD**     | password of the db user     |
|         **DB_HOST**       | database host               |

`ADMIN_USER` and `ADMIN_PASSWORD` are optional and mainly for niche purposes. Obviously, avoid clear text passwords. Once `setup.sh` has run for the first time, these variables can be removed. You should then edit `/nextcloud/config/config.php` directly if you want to change something in your configuration.

The usage of [Docker secrets](https://docs.docker.com/engine/swarm/secrets/) will be considered in the future, but `config.php` already covers quite a lot.

## Volumes

|          Variable            |         Description        |
| -------------------------    | -------------------------- |
| **/data**                    |         data files         |
| **/nextcloud/config**        |        config files        |
| **/nextcloud/apps2**         |       3rd-party apps       |
| **/nextcloud/themes**        |        custom themes       |
| **/php/session**             |      PHP session files     |

*Note: mounting `/php/session` isn't required but could be desirable in some circumstances.*

## Ports

|              Port            |            Use             |
| -------------------------    | -------------------------- |
| **8888** (tcp)               |       Nextcloud web        |

A reverse proxy like [Traefik](https://doc.traefik.io/traefik/) or [Caddy](https://caddyserver.com/) can be used, and you should consider:

- Redirecting all HTTP traffic to HTTPS
- Setting the [HSTS header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security) correctly

## Migration

From now on you'll need to make sure all volumes have proper permissions. The default UID/GID is now 1000, so you'll need to build the image yourself if you want to change that, or you can just change the actual permissions of the volumes using `chown -R 1000:1000`. The flexibility provided by the legacy image came at some cost (performance & security), therefore this feature won't be provided anymore.

Other changes that should be reflected in your configuration files:

- `/config` volume is now `/nextcloud/config`
- `/apps2` volume is now `/nextcloud/apps2`
- `ghcr.io/suprovsky/nextcloud` is the new image location

You should edit your `docker-compose.yml` and `config.php` accordingly.

## Usage

Example `docker-compose.yml`:

```docker
version: '3.9'

services:
  nextcloud:
    networks:
      - nextcloud-net
    container_name: 'nextcloud'
    depends_on:
      - nextcloud-db
    image: ghcr.io/suprovsky/nextcloud:latest
    restart: always
    ports:
      - '8888:8888'
    volumes:
      - 'nextcloud-themes:/nextcloud/themes'
      - 'nextcloud-apps:/nextcloud/apps2'
      - 'nextcloud-config:/nextcloud/config'
      - 'nextcloud-data:/data'
      - './logs:/logs'
    environment:
      - UPLOAD_MAX_SIZE=${UPLOAD_MAX_SIZE}
      - APC_SHM_SIZE=${APC_SHM_SIZE}
      - MEMORY_LIMIT=${MEMORY_LIMIT}
      - CRON_PERIOD=${CRON_PERIOD}
      - CRON_MEMORY_LIMIT={CRON_MEMORY_LIMIT}
      - DB_NAME=${DB_NAME}
      - DB_TYPE=${DB_TYPE}
      - DB_USER=${DB_USER}
      - DB_PASSWORD=${DB_PASSWORD}
      - DOMAIN=${DOMAIN}
      - PHP_HARDENING=${PHP_HARDENING}
    env_file: './.env'

  nextcloud-db:
    networks:
      - nextcloud-net
    container_name: 'nextcloud-db'
    image: rapidfort/mariadb:latest
    restart: always
    volumes:
      - 'nextcloud-db:/bitnami/mariadb'
      - './custom.cnf:/opt/bitnami/mariadb/conf/my_custom.cnf:ro'
    environment:
      - MARIADB_DATABASE=${DB_NAME}
      - MARIADB_PASSWORD=${DB_PASSWORD}
      - MARIADB_ROOT_PASSWORD=${MARIADB_ROOT_PASSWORD}
      - MARIADB_USER=${DB_USER}
    env_file: './.env'
  redis:
    container_name: nextcloud-redis
    image: rapidfort/redis:7.0
    restart: always
    environment:
      - REDIS_PASSWORD=${REDIS_PASSWORD}
    env_file:
      - ./.env
    networks:
      - nextcloud-net
    volumes:
      - nextcloud-redis:/bitnami/redis/data:rw
      - /etc/localtime:/etc/localtime:ro
      - ./redis.conf:/opt/bitnami/redis/etc/redis.conf


volumes:
  nextcloud-themes:
  nextcloud-apps:
  nextcloud-config:
  nextcloud-data:
  nextcloud-db:
  nextcloud-redis:
networks:
  nextcloud-net:
    name: nextcloud-net
```

Example `.env` file:

```env
ADMIN_USER=supersecretadminuser
ADMIN_PASSWORD=DXDXDXDXDXDXDXDXDXDXDXDXDXDXDXDXD
DB_TYPE=mysql
DB_NAME=nextcloud
DB_USER=nextclouduser
DB_PASSWORD=DXDXDXDXDXDXDXDXDXDXDXDXDXDXDXDXD
DB_HOST=nextcloud-db
UPLOAD_MAX_SIZE=50G
APC_SHM_SIZE=2G
MEMORY_LIMIT=2G
CRON_PERIOD=5m
CRON_MEMORY_LIMIT=1G
DOMAIN=nextcloud.example.com
MARIADB_ROOT_PASSWORD=DXDXDXDXDXDXDXDXDXDXDXDXDXDXDXDXD
PHP_HARDENING=true
REDIS_PASSWORD=XDXDXDXDXDXDXDXDXDXDXDXDXDXDXDXD
```

You must use some reverse proxy in front of the container. I like [Nginx](https://www.nginx.com/), so you can find an example config making use of [Let's Encrypt](https://letsencrypt.org/) certificates below.  
With this setup I got 120/100 on [Mozilla Observatory](https://observatory.mozilla.org/), A+ on [Nextcloud Security Scan](https://scan.nextcloud.com/) and A+ on [SSL Labs](https://www.ssllabs.com/).

```nginx
server {
  listen [::]:443 ssl; # managed by Certbot
  listen 443 ssl; # managed by Certbot
  ssl_certificate /etc/letsencrypt/live/nextcloud.example.com/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/nextcloud.example.com/privkey.pem; # managed by Certbot
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
  server_name nextcloud.example.com;
  access_log /var/log/nginx/nextcloud.example.com-access.log;
  error_log /var/log/nginx/nextcloud.example.com-error.log;
  ssl_session_cache shared:ssl_session_cache:10m;
  location / {
    proxy_pass http://127.0.0.1:8888$request_uri;
    proxy_hide_header X-Content-Type-Options;
    proxy_hide_header X-XSS-Protection;
    proxy_hide_header X-Robots-Tag;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Robots-Tag "noindex, nofollow" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Port $server_port;
    proxy_set_header X-Forwarded-Scheme $scheme;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Accept-Encoding "";
    proxy_set_header Host $host;
    proxy_buffering off;
    client_body_buffer_size 512k;
    proxy_read_timeout 86400s;
    client_max_body_size 0;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
  location /.well-known/carddav {
    return 301 $scheme://$host/remote.php/dav;
  }
  location /.well-known/caldav {
    return 301 $scheme://$host/remote.php/dav;
  }
}
server {
    listen 80;
    listen [::]:80;
    server_name nextcloud.example.com;
    return 302 https://nextcloud.example.com$request_uri;
}
```
