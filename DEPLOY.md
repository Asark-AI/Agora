Deployment guide — Ubuntu (Docker Compose)

Overview

This guide shows how to deploy the Next.js app to an Ubuntu server using Docker Compose and Nginx as a reverse proxy. It assumes you will run Postgres in Docker as well. Adjust as needed if you use managed Postgres.

Files added
- `Dockerfile` — production Dockerfile for building the Next.js app
- `docker-compose.prod.yml` — production compose file (app, db, nginx)
- `deployment/nginx/default.conf` — Nginx site config (proxies to app:3000)
- `.env.production.example` — example production environment file

1) Prepare the Ubuntu server

- Install Docker and Docker Compose (instructions summarised):
  - Install Docker: https://docs.docker.com/engine/install/ubuntu/
  - Install Docker Compose plugin: https://docs.docker.com/compose/install/
  - Add your deploy user to the docker group (optional):
    sudo usermod -aG docker $USER

2) Copy the repo to the server

- Clone your repo on the server or scp the files.

3) Configure environment

- Create `.env.production` in the project root (do NOT commit this file). Example:

  cp .env.production.example .env.production
  # Edit .env.production and set real secrets (DATABASE_URL, API keys, etc.)

- Important: If you keep Postgres as the `db` service in `docker-compose.prod.yml`, the default DATABASE_URL in the example points to `db` service (docker network). If you use an external DB, set `DATABASE_URL` to that host.

4) Build and start

Run:

```bash
# from project root on server
docker compose -f docker-compose.prod.yml up -d --build
```

This builds the `app` image, starts Postgres and Nginx.

5) Setup TLS with Certbot (recommended)

Install Certbot on the server and obtain certificates for your domain, or use the "standalone" or Dockerized Certbot workflows. A simple flow using Certbot + Nginx (install via snap):

```bash
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

# Stop nginx (if running) and obtain certs
sudo certbot certonly --webroot -w /var/www/certbot -d yourdomain.com -d www.yourdomain.com

# Install certs into deployment/nginx/certs and update nginx config to use them
# Example paths: /etc/letsencrypt/live/yourdomain.com/fullchain.pem
```

Alternatively, run Certbot in Docker and mount the certs into `deployment/nginx/certs` used by the nginx service.

6) Migrations and seeding (on deploy)

After the app and DB are running, run migrations & generate the Prisma client inside the app container or locally before building:

```bash
# Run in the host with docker compose
docker compose -f docker-compose.prod.yml exec app npm run prisma:generate
docker compose -f docker-compose.prod.yml exec app npm run prisma:migrate -- dev --name init
# Or for production deploy use: npx prisma migrate deploy

# Run seed (if desired)
docker compose -f docker-compose.prod.yml exec app npm run prisma:seed
```

7) Logs and maintenance

View logs:

```bash
docker compose -f docker-compose.prod.yml logs -f app
docker compose -f docker-compose.prod.yml logs -f nginx
```

8) Systemd (optional)

You can run a small systemd service that runs docker compose up on boot. Create `/etc/systemd/system/agora.service` with an ExecStart that runs `docker compose -f /path/to/docker-compose.prod.yml up -d` and enable it.

Wrap-up

This is a straightforward Docker-based production deployment. If you want, I can also create a GitHub Actions workflow that builds the image, pushes to a registry, and deploys to the server via SSH and remote docker-compose pull/up. Tell me if you'd like that next.
