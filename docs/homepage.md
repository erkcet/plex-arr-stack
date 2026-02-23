# Homepage

Homepage is a modern dashboard that shows all your services at a glance, with live status, stats from *arr APIs, and Plex activity.

## Compose Configuration

```yaml
homepage:
  image: ghcr.io/gethomepage/homepage:latest
  container_name: homepage
  environment:
    - PUID=${PUID}
    - PGID=${PGID}
  volumes:
    - ${CONFIG_DIR}/homepage:/app/config
    - /var/run/docker.sock:/var/run/docker.sock:ro
  ports:
    - "3000:3000"
  restart: unless-stopped
```

The Docker socket mount (`docker.sock`) allows Homepage to auto-discover running containers and show their status.

## First-Run Setup

1. Start the container: `docker compose up -d homepage`
2. Open `http://your-server-ip:3000`
3. You'll see the default page — configure it by editing YAML files

## Configuration Files

Homepage is configured through YAML files in `${CONFIG_DIR}/homepage/`:

| File | Purpose |
|---|---|
| `settings.yaml` | Layout, theme, title |
| `services.yaml` | Service cards and API widgets |
| `bookmarks.yaml` | Quick-link bookmarks |
| `widgets.yaml` | Top-bar system widgets |
| `docker.yaml` | Docker integration settings |

Example configs are included in this repo at `config/homepage/`.

### Copy example configs

```bash
cp config/homepage/settings.yaml ${CONFIG_DIR}/homepage/settings.yaml
cp config/homepage/services.yaml ${CONFIG_DIR}/homepage/services.yaml
```

## services.yaml

This defines the service cards on your dashboard. Each service can show live stats via API integration.

See the included [config/homepage/services.yaml](../config/homepage/services.yaml) for a full example covering all services in this stack.

Key widget types:

| Service | Widget | Shows |
|---|---|---|
| Plex | `tautulli` | Active streams, bandwidth |
| Sonarr | `sonarr` | Wanted, queued, series count |
| Radarr | `radarr` | Wanted, queued, movie count |
| Lidarr | `lidarr` | Wanted, queued, artist count |
| qBittorrent | `qbittorrent` | Active downloads, speeds |
| Prowlarr | `prowlarr` | Indexer count, recent grabs |

Each widget needs the service's URL and API key.

## settings.yaml

Controls layout and appearance. See the included [config/homepage/settings.yaml](../config/homepage/settings.yaml).

Key settings:

```yaml
title: Media Server
theme: dark
color: slate
layout:
  Media:
    columns: 1
  Management:
    columns: 3
  Downloads:
    columns: 2
  Monitoring:
    columns: 3
```

Adjust `columns` to match the number of services in each section for a clean layout.

## Docker Integration

Homepage can show container status (running/stopped) automatically.

Create `${CONFIG_DIR}/homepage/docker.yaml`:

```yaml
local:
  host: unix:///var/run/docker.sock
```

Then add `server: local` and `container: container_name` to each service in `services.yaml`.

## Customization

- **Bookmarks:** Add quick links to external tools, guides, or trackers in `bookmarks.yaml`
- **System widgets:** Show CPU, memory, disk usage in the top bar via `widgets.yaml`
- **Custom CSS:** Add a `custom.css` file for styling overrides

## Troubleshooting

**Services showing "Error":** Check the API key and URL for each widget. The URL should use the Docker container name (e.g., `http://sonarr:8989`). For Plex (host network), use your server IP.

**Changes not appearing:** Homepage watches config files and reloads automatically. If not, restart: `docker compose restart homepage`.

**Docker status not showing:** Make sure `docker.sock` is mounted and `docker.yaml` is configured.

## Next Steps

- [Networking](networking.md) — expose your dashboard securely
- [Back to main README](../README.md)
