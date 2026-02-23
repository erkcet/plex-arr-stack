# Prowlarr

Prowlarr is a centralized indexer manager. Instead of adding indexers separately to Sonarr, Radarr, and Lidarr, you add them once in Prowlarr and it syncs them to all connected apps.

## Compose Configuration

```yaml
prowlarr:
  image: lscr.io/linuxserver/prowlarr:latest
  container_name: prowlarr
  environment:
    - PUID=${PUID}
    - PGID=${PGID}
    - TZ=${TZ}
  volumes:
    - ${CONFIG_DIR}/prowlarr:/config
  ports:
    - "9696:9696"
  restart: unless-stopped
```

Prowlarr doesn't need access to media data — it only manages indexer connections.

## First-Run Setup

1. Open `http://your-server-ip:9696`
2. Set authentication: **Settings → General → Authentication → Forms (Login Page)**
3. Create a username and password

## Add Apps (Sonarr, Radarr, Lidarr)

Connect Prowlarr to the *arrs so it can sync indexers automatically.

**Settings → Apps → + → Sonarr**

| Field | Value |
|---|---|
| Sync Level | Full Sync |
| Prowlarr Server | `http://prowlarr:9696` |
| Sonarr Server | `http://sonarr:8989` |
| API Key | Sonarr's API key |

Repeat for **Radarr** (`http://radarr:7878`) and **Lidarr** (`http://lidarr:8686`).

Test each connection before saving.

## Add Indexers

**Indexers → + → Add Indexer**

Prowlarr supports hundreds of public and private indexers. Search for yours and fill in the credentials.

Common indexers:

- **Public:** 1337x, RARBG (if available), The Pirate Bay, YTS
- **Private:** Check your tracker's wiki for Prowlarr/Torznab compatibility

After adding an indexer, Prowlarr automatically syncs it to all connected apps.

### Tags for Selective Sync

If you want certain indexers only for specific apps:

1. Create a tag in Prowlarr (e.g., `movies-only`)
2. Add the tag to the indexer
3. Add the same tag to the Radarr app connection

Only indexers with matching tags (or no tags) will sync to that app.

## Download Client Integration

Prowlarr can also link to your download client for manual searches from the Prowlarr UI:

**Settings → Download Clients → + → qBittorrent**

| Field | Value |
|---|---|
| Host | `qbittorrent` |
| Port | `8080` |
| Username | your qBit username |
| Password | your qBit password |

## How Sync Works

```
Prowlarr                    Sonarr
┌──────────┐    sync     ┌──────────┐
│ Indexer A │───────────►│ Indexer A │
│ Indexer B │───────────►│ Indexer B │
│ Indexer C │            └──────────┘
└──────┬───┘
       │        sync      Radarr
       └────────────────►┌──────────┐
                         │ Indexer A │
                         │ Indexer B │
                         │ Indexer C │
                         └──────────┘
```

- **Full Sync** — adds, removes, and updates indexers in the connected app
- **Add and Remove Only** — won't update existing indexer settings
- **Disabled** — no sync (manual only)

## Troubleshooting

**Sync not working:** Check the app connections — ensure the API keys are correct and Prowlarr can reach the services (test the connection).

**Indexer errors:** Click the indexer name to see recent query results and errors. Many issues are from incorrect credentials or the site being down.

**Indexers not appearing in Sonarr/Radarr:** Make sure the sync level is "Full Sync" and there are no tag mismatches.

## Next Steps

- [Sonarr](sonarr.md) — TV show management
- [Radarr](radarr.md) — movie management
- [Lidarr](lidarr.md) — music management
- [Back to main README](../README.md)
