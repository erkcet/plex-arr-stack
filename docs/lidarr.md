# Lidarr

Lidarr automates music downloads — it monitors artists, searches indexers, and organizes your music library.

## Compose Configuration

```yaml
lidarr:
  image: lscr.io/linuxserver/lidarr:latest
  container_name: lidarr
  environment:
    - PUID=${PUID}
    - PGID=${PGID}
    - TZ=${TZ}
  volumes:
    - ${CONFIG_DIR}/lidarr:/config
    - ${DATA_DIR}:/data
  ports:
    - "8686:8686"
  restart: unless-stopped
```

## First-Run Setup

1. Open `http://your-server-ip:8686`
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
| Category | `music` |

## Set Root Folder

**Settings → Media Management → Root Folders → Add Root Folder**

- Path: `/data/media/music`

## Quality Profiles

Lidarr's default profiles work for most users. Key options:

- **Lossless** — FLAC, prefers highest quality
- **Standard** — MP3-320 / AAC, smaller files
- **Any** — grabs whatever is available first

For Plex music libraries, FLAC is recommended if storage isn't a concern.

## Metadata Profiles

Lidarr uses metadata profiles to decide which albums to monitor:

- **Standard** — studio albums only
- **Everything** — includes singles, EPs, compilations, live albums

Start with **Standard** unless you want everything from an artist.

**Settings → Profiles → Metadata Profiles**

## Import Existing Library

1. Go to **Artist → Import Existing**
2. Point to `/data/media/music`
3. Lidarr will attempt to match folders to artists — review and confirm

> Lidarr expects a folder-per-artist structure: `/data/media/music/Artist Name/Album Name/`

## Recommended Settings

**Settings → Media Management:**

- **Rename Tracks:** Yes
- **Standard Track Format:** `{Artist CleanName} - {Album Title} - {track:00} - {Track CleanTitle}`
- **Artist Folder Format:** `{Artist CleanName}`

## Connecting Prowlarr

Indexers are synced from Prowlarr. See [prowlarr.md](prowlarr.md).

> Note: Music indexers can be tricky. Not all indexers have good music coverage. Headphones-compatible indexers and music-specific trackers work best.

## Troubleshooting

**Artist not found:** Lidarr uses MusicBrainz for metadata. If an artist isn't in MusicBrainz, Lidarr can't track them. You can add missing artists to MusicBrainz.

**Wrong albums imported:** Check your metadata profile — you may have "Everything" selected, pulling in unwanted releases.

**Downloads fail:** Music torrents are less common than TV/movies. You may need specialized indexers.

## Next Steps

- [Prowlarr](prowlarr.md) — manage indexers centrally
- [Back to main README](../README.md)
