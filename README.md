# Portainer Compose

Templates for managing Docker stacks with Portainer. Each directory is one independent stack.

## Layout

```text
infra/
├── admin/          # termix, guacd, dbx, redisinsight
├── dashboard/      # sun-panel, flatnas
├── data/           # mariadb, postgres, redis, meilisearch
├── maintenance/    # watchtower (auto image updates)
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
├── maintenance/   # watchtower (auto image updates)
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

## Automatic image updates

`infra/` and `media/` are two separate Docker hosts, so each one runs its own Watchtower
instance from `<host>/maintenance/`. Watchtower cannot manage more than one Docker daemon
per instance (`--host` is a single value).

Both instances run in opt-in mode (`WATCHTOWER_LABEL_ENABLE=true`): a container is only
auto-updated when it carries the label

```yaml
labels:
  - com.centurylinklabs.watchtower.enable=true
```

A global `WATCHTOWER_COOLDOWN_DELAY=24h` applies on both hosts, so an image pushed
tonight is picked up by the run after tomorrow's, not the same night.

Currently opted in:

| Host | Services |
| --- | --- |
| `infra/` | `admin/termix`, `admin/dbx`, `admin/redisinsight`, `notes/joplin`, `security/vaultwarden` |
| `media/` | `ai/magpie`, `photos/immich`, `library/navidrome`, `library/audiobookshelf`, `library/calibre-web`, `media/jellyfin`, `media-tools/tinymediamanager`, `media-tools/metatube`, `media-tools/atvloadly`, `tools/it-tools` |

Per-container `com.centurylinklabs.watchtower.cooldown-delay` labels are reserved for
exceptions and currently none are set, since the global value already covers the common
case. Use them to deviate from it:

- `1w` — a longer buffer for migration-heavy services.
- `0` — opt a single container back into immediate updates.

The cooldown only delays *new* images: Watchtower still compares digests first, so it
never adds registry traffic for containers that are already current. When an image is
too fresh it is reported as `Skipped` with an `eligible_at` timestamp instead of failing.

The label is an opt-in marker, not a guarantee: it only takes effect while the service
keeps a floating tag. Several services are still pinned and are deliberately **not**
labelled, because their tag never moves:
`admin/guacd:1.6.0`, `dashboard/sun-panel:1.3.0`, `downloads/fluxdown-server:0.4.3`,
`custom/vidlens-server:1.0.1`, `custom/telegram-assistant:1.0.0`,
`games/emulatorjs:amd64-1.9.2`.

Rules of thumb when adding the label:

- The image tag must be floating (`latest`, `1`, `16`, ...). A pinned tag such as
  `sun-panel:1.3.0` never changes, so the label does nothing.
- Do not label stateful or irreversible services: `infra/data/*`, `media/cloud/nextcloud`,
  `media/photos` DB, `media/photos/photoprism`, `infra/notes/trilium`, `media/ai/open-webui`,
  `media/ai/n8n`, `media/downloads/xunlei`, `media/cloud/openlist`. They migrate data or
  hold sessions on startup and have no rollback path.
- `WATCHTOWER_ROLLING_RESTART` stays `false` in both instances. Watchtower validates the
  whole filtered set at startup and exits non-zero if *any* managed container has a
  dependency (`com.docker.compose.depends_on`, links, network mode, `volumes_from`).
  `immich` has `depends_on`, so enabling rolling restarts would make the instance
  crash-loop instead of updating anything.
