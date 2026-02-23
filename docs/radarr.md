# Radarr

Radarr automates movie downloads — it searches indexers, sends to your download client, and organizes the files into your library.

## Compose Configuration

```yaml
radarr:
  image: lscr.io/linuxserver/radarr:latest
  container_name: radarr
  environment:
    - PUID=${PUID}
    - PGID=${PGID}
    - TZ=${TZ}
  volumes:
    - ${CONFIG_DIR}/radarr:/config
    - ${DATA_DIR}:/data
  ports:
    - "7878:7878"
  restart: unless-stopped
```

## First-Run Setup

1. Open `http://your-server-ip:7878`
2. Set authentication: **Settings → General → Authentication → Forms (Login Page)**
3. Create a username and password
4. Note the **API Key** — needed for Prowlarr, Unpackerr, and Notifiarr

## Connect Download Client

**Settings → Download Clients → + → qBittorrent**

| Field | Value |
|---|---|
| Host | `qbittorrent` |
| Port | `8080` |
| Username | your qBit username |
| Password | your qBit password |
| Category | `movies` |

## Set Root Folder

**Settings → Media Management → Root Folders → Add Root Folder**

- Path: `/data/media/movies`

## Quality Profiles

For optimized profiles, follow TRaSH Guides:

- [TRaSH — Radarr Quality Profiles](https://trash-guides.info/Radarr/)

Recommended starting profile: **HD-1080p** or **HD Bluray + WEB** depending on your preference.

## Import Existing Library

1. Go to **Movies → Import Existing Movies**
2. Point to `/data/media/movies`
3. Radarr matches folders to movies — review and confirm

## Recommended Settings

**Settings → Media Management:**

- **Rename Movies:** Yes
- **Standard Movie Format:** `{Movie CleanTitle} {(Release Year)} [imdbid-{ImdbId}] - {Edition Tags} [{Quality Full}]{[MediaInfo VideoDynamicRangeType]}`
- **Movie Folder Format:** `{Movie CleanTitle} ({Release Year})`
- **Create empty folders:** Yes
- **Delete empty folders:** Yes

**Settings → Custom Formats:**

TRaSH Guides provides import-ready custom formats for scoring releases. These are highly recommended for getting the best quality releases:

- [TRaSH — Radarr Custom Formats](https://trash-guides.info/Radarr/Radarr-collection-of-custom-formats/)

## Connecting Prowlarr

Indexers are synced from Prowlarr automatically. See [prowlarr.md](prowlarr.md).

## Troubleshooting

**Movie stuck at "Downloading":** Check qBittorrent — is the torrent actually downloading? Check the category is `movies`.

**Wrong movie matched:** Click the movie → Edit → change the TMDB ID manually.

**Quality upgrades not happening:** Make sure your quality profile has "Upgrades Allowed" enabled and the upgrade cutoff is set higher than the current quality.

## Next Steps

- [Lidarr](lidarr.md) — music management
- [Prowlarr](prowlarr.md) — manage indexers centrally
- [Back to main README](../README.md)
