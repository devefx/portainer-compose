# Portainer Compose

Templates for managing Docker stacks with Portainer. Each directory is one independent stack.

## Layout

```text
infra/
├── admin/          # termix, guacd, dbx, redisinsight
├── dashboard/      # sun-panel, flatnas
├── data/           # mariadb, postgres, redis, meilisearch
├── metrics/        # grafana, influxdb2, prometheus
├── notes/          # joplin, trilium
├── notifications/  # ntfy
└── security/       # vaultwarden

media/
├── photos/        # immich, photoprism
├── cloud/         # nextcloud, openlist
├── media/         # emby, jellyfin
├── library/       # navidrome, audiobookshelf, calibre-web
├── media-tools/   # tinymediamanager, metatube, atvloadly
├── downloads/     # fluxdown-server, xunlei
├── ai/            # open-webui, n8n
├── games/         # emulatorjs, couple-flying-chess
├── smart-home/    # home-assistant (reserved)
├── tools/         # it-tools
└── custom/        # vidlens-server, telegram-assistant
```

## Usage

1. Copy `.env.example` to `.env` in the stack directory.
2. Fill in real values. Never commit `.env`.
3. Deploy locally with `docker compose up -d` from the stack directory.
4. In Portainer, create one stack per directory and set the compose path to `infra/<stack>/docker-compose.yml`.

## Environment variables

- `.env.example` documents every key used by the compose file.
- For Portainer Git-based stacks, do not rely on automatic `.env` loading from subdirectories. Use the Portainer stack editor or the "Load variables from .env file" option instead.
