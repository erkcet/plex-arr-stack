# Sonarr

Sonarr automates TV show downloads — it monitors for new episodes, searches indexers, sends to your download client, and organizes the files into your library.

## Compose Configuration

```yaml
sonarr:
  image: lscr.io/linuxserver/sonarr:latest
  container_name: sonarr
  environment:
    - PUID=${PUID}
    - PGID=${PGID}
    - TZ=${TZ}
  volumes:
    - ${CONFIG_DIR}/sonarr:/config
    - ${DATA_DIR}:/data
  ports:
    - "8989:8989"
  restart: unless-stopped
```

The entire `${DATA_DIR}` is mounted as `/data` so hardlinks work between `/data/torrents/tv` and `/data/media/tv`.

## First-Run Setup

1. Open `http://your-server-ip:8989`
2. Set authentication: **Settings → General → Authentication → Forms (Login Page)**
3. Create a username and password
4. Note the **API Key** on the General page — you'll need it for Prowlarr, Unpackerr, and Notifiarr

## Connect Download Client

**Settings → Download Clients → + → qBittorrent**

| Field | Value |
|---|---|
| Host | `qbittorrent` |
| Port | `8080` |
| Username | your qBit username |
| Password | your qBit password |
| Category | `tv` |

Test the connection, then save.

## Set Root Folder

**Settings → Media Management → Root Folders → Add Root Folder**

- Path: `/data/media/tv`

## Quality Profiles

Sonarr ships with default profiles. For optimized quality settings, follow the TRaSH Guides:

- [TRaSH — Sonarr Quality Profiles](https://trash-guides.info/Sonarr/)

At minimum:

- **Settings → Quality:** Adjust size limits per quality if needed
- **Settings → Profiles:** Select or customize a quality profile (e.g., "HD-1080p" for most users)

## Import Existing Library

If you already have TV shows on disk:

1. Go to **Series → Import Existing Series**
2. Point to `/data/media/tv`
3. Sonarr will match folders to series — review and confirm

## Recommended Settings

**Settings → Media Management:**

- **Rename Episodes:** Yes
- **Standard Episode Format:** `{Series TitleYear} - S{season:00}E{episode:00} - {Episode CleanTitle} [{Quality Full}]{[MediaInfo VideoDynamicRangeType]}`
- **Season Folder Format:** `Season {season:00}`
- **Create empty series folders:** Yes
- **Delete empty folders:** Yes

**Settings → Indexers:**

Indexers are managed by Prowlarr — see [prowlarr.md](prowlarr.md). After Prowlarr syncs, they'll appear here automatically.

## Connecting Prowlarr

Prowlarr pushes indexers to Sonarr automatically. You don't need to add indexers manually in Sonarr. See [prowlarr.md](prowlarr.md).

## Troubleshooting

**Downloads stuck at "Importing":** Check that the download category in qBittorrent matches what Sonarr expects (`tv`). Check file permissions.

**Hardlinks not working:** Verify that both `/data/torrents/tv` and `/data/media/tv` are under the same Docker volume mount. See [storage.md](storage.md).

**Series not matching:** Use the search bar to manually search and map a series to the correct TVDB entry.

## Next Steps

- [Radarr](radarr.md) — same workflow, but for movies
- [Prowlarr](prowlarr.md) — manage indexers centrally
- [Back to main README](../README.md)
