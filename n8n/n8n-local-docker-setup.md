# n8n Local Docker Setup (macOS)

## Prerequisites

-   Docker Desktop installed and running
-   Terminal access

------------------------------------------------------------------------

## Directory Structure

``` text
n8n/
├── compose.yml
├── .env
└── data/
```

------------------------------------------------------------------------

## `compose.yml`

``` yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n

    restart: unless-stopped

    ports:
      - "127.0.0.1:5678:5678"

    environment:
      - TZ=${TZ}

      - N8N_HOST=${N8N_HOST}
      - N8N_PORT=5678
      - N8N_PROTOCOL=https

      - WEBHOOK_URL=${WEBHOOK_URL}

      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}

      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD}

      - GENERIC_TIMEZONE=${TZ}

      - N8N_DIAGNOSTICS_ENABLED=false
      - N8N_VERSION_NOTIFICATIONS_ENABLED=false
      - N8N_PERSONALIZATION_ENABLED=false

    volumes:
      - ./data:/home/node/.n8n

    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:5678/healthz"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 30s
```

------------------------------------------------------------------------

## `.env`

``` env
TZ=Asia/Dhaka

N8N_HOST=n8n.example.com
WEBHOOK_URL=https://n8n.example.com/

N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=ChooseAVeryStrongPassword

N8N_ENCRYPTION_KEY=replace-with-at-least-32-random-characters
```

Generate an encryption key:

``` bash
openssl rand -hex 32
```

------------------------------------------------------------------------

## Start

``` bash
docker compose up -d
```

------------------------------------------------------------------------

## Verify

``` bash
docker ps
```

Open:

``` text
http://localhost:5678
```

Sign in using the Basic Auth credentials from the `.env` file.

------------------------------------------------------------------------

## Restart

``` bash
docker compose restart
```

------------------------------------------------------------------------

## Stop

``` bash
docker compose down
```

------------------------------------------------------------------------

## Notes

-   Data, credentials, and workflows are stored in the `data/`
    directory.
-   The service is only accessible from the local machine (`127.0.0.1`).
-   This setup uses SQLite, which is suitable for local development.
