# Storage & Directory Structure

Getting the directory structure right is critical. A proper layout enables **hardlinks** — meaning files are imported instantly with zero extra disk space.

## Why This Matters

When Sonarr/Radarr "imports" a download, it can either:

| Method | Speed | Extra Disk Space | Seeding |
|---|---|---|---|
| **Hardlink** | Instant | None | Continues |
| Copy | Slow | 2x per file | Continues |
| Move | Fast | None | **Breaks** |

Hardlinks only work when source and destination are on the **same filesystem**. That's why both `torrents/` and `media/` must live under one mount point.

## Recommended Structure

```
/data                       ← single mount point (one filesystem)
├── torrents/               ← qBittorrent downloads here
│   ├── movies/
│   ├── tv/
│   └── music/
└── media/                  ← organized library (Plex reads this)
    ├── movies/
    ├── tv/
    └── music/
```

This maps to the `DATA_DIR` variable in your `.env`.

## Creating the Directories

```bash
# Set this to match your .env
DATA_DIR=/opt/plex-arr-stack/data

sudo mkdir -p "$DATA_DIR"/{media/{movies,tv,music},torrents/{movies,tv,music}}
sudo chown -R $(id -u):$(id -g) "$DATA_DIR"
```

## How Services See the Paths

Every service that needs data access mounts `${DATA_DIR}` as `/data` inside the container:

```yaml
volumes:
  - ${DATA_DIR}:/data
```

So inside the container:

| Host Path | Container Path |
|---|---|
| `$DATA_DIR/torrents/movies` | `/data/torrents/movies` |
| `$DATA_DIR/torrents/tv` | `/data/torrents/tv` |
| `$DATA_DIR/media/movies` | `/data/media/movies` |
| `$DATA_DIR/media/tv` | `/data/media/tv` |

Plex only needs the media directory, so it mounts just `${DATA_DIR}/media:/data/media`.

## Hardlink Requirements

For hardlinks to work:

1. **Same filesystem** — `torrents/` and `media/` must be on the same partition/disk/pool.
2. **Same mount in Docker** — both paths must be under one volume mount (`/data`). Do **not** mount them as separate volumes.
3. **Correct permissions** — all containers run as the same PUID/PGID.

### Verifying hardlinks work

After a download is imported, check if it's a hardlink:

```bash
# If link count > 1, it's hardlinked
ls -l /opt/plex-arr-stack/data/media/movies/SomeMovie/SomeMovie.mkv

# Or use stat
stat /opt/plex-arr-stack/data/media/movies/SomeMovie/SomeMovie.mkv
# Look for "Links: 2" (or higher)
```

## Config Directory

Service configs are stored separately from media data:

```
/opt/plex-arr-stack/config/    ← CONFIG_DIR in .env
├── plex/
├── sonarr/
├── radarr/
├── lidarr/
├── prowlarr/
├── qbittorrent/
├── autobrr/
├── bazarr/
├── tautulli/
├── notifiarr/
└── homepage/
```

These are created automatically when each container starts for the first time.

## Multiple Disks

If your media spans multiple disks, you have options:

- **MergerFS** — merges multiple disks into a single mount point. Hardlinks work across merged paths.
- **ZFS / RAID** — presents multiple disks as one pool.
- **Separate mount points** — works, but hardlinks won't work across different filesystems (the *arrs will copy instead).

## Permissions Checklist

```bash
# Everything should be owned by your user
ls -la /opt/plex-arr-stack/data/
# drwxr-xr-x  youruser youruser  media/
# drwxr-xr-x  youruser youruser  torrents/

# If permissions are wrong:
sudo chown -R $(id -u):$(id -g) /opt/plex-arr-stack/data/
sudo chmod -R 755 /opt/plex-arr-stack/data/
```

## Next Steps

- [qBittorrent setup](qbittorrent.md) — configure download paths and categories
- [Back to main README](../README.md)
