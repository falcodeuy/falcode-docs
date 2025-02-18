---
layout: default
title: Site certificates
parent: Project configuration
grand_parent: Django
nav_order: 2
---

# Certificates

This document describes how to add certificates to your site using `certbot`, `docker`, and `nginx`. For this process, the site must be already running on `HTTP`.

## Step 1: Edit docker compose file

Edit your docker compose file to add the `certbot` service:

```yaml
certbot:
  image: certbot/certbot:latest
  volumes:
    - ./certbot/www/:/var/www/certbot/:rw
    - ./certbot/conf/:/etc/letsencrypt/:rw
```

You also need to add these two lines as new volumes to the `nginx` service to ensure it has access to the certificates. The compose file with both services should look like this:

```yaml
services:
  webserver:
    image: nginx:latest
    ports:
      - 80:80
      - 443:443
    restart: always
    volumes:
      - ./nginx/conf/:/etc/nginx/conf.d/:ro
      # Certbot configuration
      - ./certbot/www:/var/www/certbot/:ro
      - ./certbot/conf/:/etc/nginx/ssl/:ro
  certbot:
    image: certbot/certbot:latest
    volumes:
      - ./certbot/www/:/var/www/certbot/:rw
      - ./certbot/conf/:/etc/letsencrypt/:rw
```

Now, run `docker compose --build -d` to install the `certbot` service. This command only installs the service; the certificate configuration is added in the next steps.

## Step 2: Nginx http configuration

In your `nginx` configuration file, you already have some server configuration on port `80` for `HTTP`. We need to do two things now, add the following location to this server configuration:

```nginx
location /.well-known/acme-challenge/ {
    root /var/www/certbot;
}
```

This step is necessary for `certbot` configuration. Then, you need to add, or replace if you already have it, the root location to this:

```nginx
location / {
    return 301 https://[domain-name]$request_uri;
}
```

This tell `nginx` to redirect the traffic from `HTTP` to `HTTPS`. The final configuration should look something like this:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name [domain-name] www.[domain-name];
    server_tokens off;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://[domain-name]$request_uri;
    }
}
```

Remember to reload `nginx` to apply the new configuration. Uou can run `docker compose restart` to restart all services or just `docker compose exec webserver nginx -s reload` to reload `nginx` without interrupting other services.

If you already configured some `https` (server listening on port `433`) comment out that part temporarily to avoid errors during the certificate creation process.

## Step 3: Create certificates

First of all, ensure all configurations are ready because `certbot` has limited retries for the configuration command. If the command fails multiple times, it will be blocked for a certain period.

To test the configuration command, use it with the `--dry-run` flag, replacing your domain name at the end:

```bash
docker compose -f your_compose_file.yml run --rm  certbot certonly --webroot --webroot-path /var/www/certbot/ --dry-run -d [domain-name]
```

If you receive a message stating "The dry run was successful", you can proceed. Otherwise, review the previous steps or check your DNS configuration.

Now, run `certbot` without `--dry-run` flag to configure the new certificates:

```bash
docker compose -f your_compose_file.yml run --rm certbot certonly --webroot --webroot-path /var/www/certbot/  -d [domain-name]
```

Follow the instructions from the command prompt, which will request an email (this can be your personal email) and ask you to accept the terms and conditions.

Now add the `HTTPS` configuration for `nginx` and restart the service:

```nginx
server {
    listen 443 default_server ssl http2;
    listen [::]:443 ssl http2;

    server_name [domain-name];

    ssl_certificate /etc/nginx/ssl/live/[domain-name]/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/live/[domain-name]/privkey.pem;

    location / {
    	# Django configuration
    }
}
```

Remember to replace the domain name and restart `nginx` service `docker compose webserver restart`.

At the end, your nginx configuration file should look like this:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name [domain-name];
    server_tokens off;


    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://[domain-name]$request_uri;
    }
}

server {
    listen 443 default_server ssl;
    listen [::]:443 ssl;

    client_max_body_size 256M;
    server_name [domain-name];

    ssl_certificate /etc/nginx/ssl/live/[domain-name]/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/live/[domain-name]/privkey.pem;

    location / {
        proxy_pass http://[django_service_name]:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /static/ {
        alias /code/static/;
    }
}
```

# Step 4: Renew certificates

To manually renew certificates, you can run:

```bash
docker compose run --rm certbot renew
```

To automatically renew them periodically, say every 60 days, you can use `crontab`. Open the `crontab` configuration with:

```bash
contrab -e
```

Now, add the following line, replacing your `Docker` compose file path:

```bash
0 5 1 */2 *  /usr/bin/docker compose -f /home/ubuntu/repository/docker-compose.prod.yml run --rm certbot renew --force-renew && /usr/bin/docker compose -f /home/ubuntu/repository/docker-compose.prod.yml restart nginx
```

Ensure `/usr/bin/docker` exists and your path to the Docker file is correct, this command also restart `nginx`, if you are using other service replace the command after `&&`. Save the file. This will run the renewal process on the first day at 5 AM every two months.
