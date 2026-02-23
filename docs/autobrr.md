# Autobrr

Autobrr monitors IRC announce channels and RSS feeds for new releases, grabbing them the instant they're available. It can send grabs directly to qBittorrent or route them through Sonarr/Radarr for full automation.

## Compose Configuration

```yaml
autobrr:
  image: ghcr.io/autobrr/autobrr:latest
  container_name: autobrr
  environment:
    - TZ=${TZ}
  user: "${PUID}:${PGID}"
  volumes:
    - ${CONFIG_DIR}/autobrr:/config
  ports:
    - "7474:7474"
  restart: unless-stopped
```

Note: Autobrr uses `user:` instead of `PUID`/`PGID` environment variables.

## First-Run Setup

1. Open `http://your-server-ip:7474`
2. Create an account (first-run setup wizard)
3. Log in

## Add Download Client

**Settings → Download Clients → Add**

| Field | Value |
|---|---|
| Type | qBittorrent |
| Host | `qbittorrent` |
| Port | `8080` |
| Username | your qBit username |
| Password | your qBit password |

## Add Indexers (IRC)

**Settings → Indexers → Add**

Select your tracker from the list. You'll need:

- IRC server, port, and channel (usually provided by the tracker)
- Your IRC nick and NickServ password (or invite command)
- Announce key / passkey from the tracker

Autobrr supports many trackers out of the box — check the [supported indexers list](https://autobrr.com/configuration/indexers).

## Create Filters

Filters define what to grab. Go to **Filters → Add**.

Example: grab all 1080p BluRay movie releases:

| Field | Value |
|---|---|
| Name | `1080p BluRay Movies` |
| Resolution | `1080p` |
| Source | `BluRay` |
| Match Categories | (tracker-specific) |

### Actions

Each filter needs an action — what to do when a match is found:

- **Send to qBittorrent** — direct grab, bypasses the *arrs
- **Send to Sonarr/Radarr** — pushes to the *arr, which handles import and organization (recommended)

For Sonarr/Radarr actions:

| Field | Value |
|---|---|
| Type | Sonarr / Radarr |
| Host | `http://sonarr:8989` or `http://radarr:7878` |
| API Key | the app's API key |

## Use Cases

- **Race for fresh uploads** — grab releases within seconds of being announced
- **Hard-to-find content** — set filters for specific titles, groups, or quality
- **Freeleech** — grab freeleech-tagged releases automatically on private trackers

## Autobrr vs Prowlarr

| Feature | Prowlarr | Autobrr |
|---|---|---|
| Search type | On-demand (API search) | Real-time (IRC announce) |
| Speed | Minutes | Seconds |
| Use case | General automation | Racing, niche grabs |
| Complexity | Low | Medium |

They complement each other — Prowlarr handles regular searches, Autobrr handles instant grabs.

## Troubleshooting

**IRC not connecting:** Check your IRC credentials. Some trackers require you to register your nick before connecting. Check the Autobrr logs: `docker logs autobrr`.

**Filter not matching:** Use the "Test" feature on your filter to check against recent announces. Make sure your match criteria aren't too restrictive.

## Next Steps

- [Unpackerr](unpackerr.md) — auto-extract archived downloads
- [Back to main README](../README.md)
