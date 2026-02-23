# plex-arr-stack

A complete Docker Compose media server stack — Plex, Sonarr, Radarr, Lidarr, Prowlarr, qBittorrent, and useful extras. One command to deploy, fully configurable via `.env`.

**Target audience:** Users comfortable with Linux basics and Docker who want a turnkey media automation setup.

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         INDEXERS (Usenet / Torrent)                  │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                        ┌──────▼──────┐
                  ┌─────│  Prowlarr   │─────┐
                  │     │  :9696      │     │
                  │     └──────┬──────┘     │
                  │            │             │
           ┌──────▼──────┐ ┌──▼───────┐ ┌──▼───────┐
           │   Sonarr    │ │  Radarr  │ │  Lidarr  │
           │   :8989     │ │  :7878   │ │  :8686   │
           │  (TV Shows) │ │ (Movies) │ │ (Music)  │
           └──────┬──────┘ └────┬─────┘ └────┬─────┘
                  │             │             │
                  └──────┬──────┘─────────────┘
                         │
                  ┌──────▼──────┐     ┌──────────────┐
                  │ qBittorrent │     │   Autobrr    │
                  │   :8080     │◄────│   :7474      │
                  └──────┬──────┘     └──────────────┘
                         │
              ┌──────────┼──────────┐
              │          │          │
       ┌──────▼───┐ ┌───▼────┐ ┌──▼───────┐
       │Unpackerr │ │ Bazarr │ │   Plex   │
       │(extract) │ │ :6767  │ │  :32400  │
       └──────────┘ │(subs)  │ └──────────┘
                    └────────┘      │
                               ┌────▼─────┐
                               │ Tautulli │
                               │  :8181   │
                               └──────────┘

        ┌────────────┐    ┌────────────┐
        │ Notifiarr  │    │  Homepage  │
        │   :5454    │    │   :3000    │
        └────────────┘    └────────────┘
```

### Data flow

1. **Prowlarr** manages indexer connections and syncs them to Sonarr, Radarr, and Lidarr.
2. **Sonarr / Radarr / Lidarr** search for media, send download requests to qBittorrent.
3. **Autobrr** monitors IRC announce channels for instant grabs, sending to qBittorrent or the *arrs.
4. **qBittorrent** downloads torrents to `/data/torrents/{tv,movies,music}`.
5. **Unpackerr** extracts archived downloads so the *arrs can import them.
6. The *arrs **hardlink** (or copy) completed downloads into `/data/media/{tv,movies,music}`.
7. **Plex** serves the organized media library to your devices.
8. **Bazarr** fetches subtitles for content in Sonarr and Radarr.
9. **Tautulli** monitors Plex activity and history.
10. **Notifiarr** sends notifications (Discord, Telegram, etc.) from all services.
11. **Homepage** provides a dashboard with service status and quick links.

## Services

| Service | Port | Purpose |
|---|---|---|
| Plex | 32400 | Media server |
| Sonarr | 8989 | TV show management |
| Radarr | 7878 | Movie management |
| Lidarr | 8686 | Music management |
| Prowlarr | 9696 | Indexer management |
| qBittorrent | 8080 | Torrent downloads |
| Autobrr | 7474 | IRC autodl / filters |
| Unpackerr | — | Auto-extract archives |
| Bazarr | 6767 | Subtitle management |
| Tautulli | 8181 | Plex monitoring |
| Notifiarr | 5454 | Notifications hub |
| Homepage | 3000 | Dashboard |

## Quick Start

### 1. Prerequisites

Make sure you have Docker and Docker Compose installed. See [docs/prerequisites.md](docs/prerequisites.md) for details.

### 2. Clone and configure

```bash
git clone https://github.com/yourusername/plex-arr-stack.git
cd plex-arr-stack

# Copy the example env file and edit it
cp .env.example .env
nano .env
```

At minimum, set these values in `.env`:

- `PUID` / `PGID` — run `id` to find yours
- `TZ` — your timezone
- `CONFIG_DIR` — where service configs will be stored
- `DATA_DIR` — your root data directory (see [docs/storage.md](docs/storage.md))
- `PLEX_CLAIM` — get a token at [plex.tv/claim](https://plex.tv/claim)

### 3. Create the directory structure

```bash
# Create the data directories (adjust DATA_DIR to match your .env)
DATA_DIR=/opt/plex-arr-stack/data

sudo mkdir -p "$DATA_DIR"/{media/{movies,tv,music},torrents/{movies,tv,music}}
sudo chown -R $(id -u):$(id -g) "$DATA_DIR"
```

### 4. Start the stack

```bash
docker compose up -d
```

### 5. Configure services

Follow the setup guides in order:

1. [qBittorrent](docs/qbittorrent.md) — download client
2. [Prowlarr](docs/prowlarr.md) — indexer management
3. [Sonarr](docs/sonarr.md) — TV shows
4. [Radarr](docs/radarr.md) — movies
5. [Lidarr](docs/lidarr.md) — music
6. [Plex](docs/plex.md) — media server
7. [Bazarr](docs/bazarr.md) — subtitles
8. [Autobrr](docs/autobrr.md) — IRC autodl
9. [Unpackerr](docs/unpackerr.md) — auto-extract
10. [Tautulli](docs/tautulli.md) — monitoring
11. [Notifiarr](docs/notifiarr.md) — notifications
12. [Homepage](docs/homepage.md) — dashboard

## Disabling Services

Don't need everything? Comment out any service block in `docker-compose.yml`:

```yaml
  # bazarr:
  #   image: lscr.io/linuxserver/bazarr:latest
  #   ...
```

Then run `docker compose up -d` again to apply.

## Documentation

| Guide | Description |
|---|---|
| [Prerequisites](docs/prerequisites.md) | Hardware recs, OS setup, Docker install |
| [Storage](docs/storage.md) | Directory structure, permissions, hardlinks |
| [Networking](docs/networking.md) | Reverse proxy, Cloudflare tunnels, remote access |
| [Maintenance](docs/maintenance.md) | Backups, updates, logs, troubleshooting |

## Further Reading

- [TRaSH Guides](https://trash-guides.info/) — quality profiles, naming conventions, and best practices
- [LinuxServer.io](https://docs.linuxserver.io/) — container documentation
- [Servarr Wiki](https://wiki.servarr.com/) — official *arr documentation
