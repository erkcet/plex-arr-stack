# Unpackerr

Unpackerr monitors your download directories and automatically extracts archived files (RAR, ZIP, etc.) so the *arrs can import them. Many releases, especially from private trackers, come as compressed archives.

## Compose Configuration

```yaml
unpackerr:
  image: ghcr.io/unpackerr/unpackerr:latest
  container_name: unpackerr
  environment:
    - TZ=${TZ}
    # Sonarr
    - UN_SONARR_0_URL=http://sonarr:8989
    - UN_SONARR_0_API_KEY=${SONARR_API_KEY}
    - UN_SONARR_0_PATHS_0=/data/torrents/tv
    # Radarr
    - UN_RADARR_0_URL=http://radarr:7878
    - UN_RADARR_0_API_KEY=${RADARR_API_KEY}
    - UN_RADARR_0_PATHS_0=/data/torrents/movies
    # Lidarr
    - UN_LIDARR_0_URL=http://lidarr:8686
    - UN_LIDARR_0_API_KEY=${LIDARR_API_KEY}
    - UN_LIDARR_0_PATHS_0=/data/torrents/music
  volumes:
    - ${DATA_DIR}/torrents:/data/torrents
  restart: unless-stopped
```

Unpackerr is configured entirely through environment variables — no web UI.

## How It Works

1. Sonarr/Radarr/Lidarr report that a download is complete but contains archives
2. Unpackerr detects the queued item via the *arr API
3. It extracts the archive into the same directory
4. The *arr picks up the extracted files and imports them
5. Unpackerr cleans up extracted files after a configurable delay

## Setup

Unpackerr requires API keys from each *arr it monitors. After starting Sonarr, Radarr, and Lidarr for the first time:

1. Get each service's API key (Settings → General)
2. Add them to your `.env` file:

   ```
   SONARR_API_KEY=your_key_here
   RADARR_API_KEY=your_key_here
   LIDARR_API_KEY=your_key_here
   ```

3. Restart Unpackerr: `docker compose restart unpackerr`

## Configuration Options

All options are set via environment variables. Common ones:

| Variable | Default | Description |
|---|---|---|
| `UN_DEBUG` | `false` | Enable debug logging |
| `UN_INTERVAL` | `2m` | How often to check for extractions |
| `UN_START_DELAY` | `1m` | Wait time after container start |
| `UN_RETRY_DELAY` | `5m` | Wait time between retries on failure |
| `UN_MAX_RETRIES` | `3` | Max extraction attempts |
| `UN_PARALLEL` | `1` | Concurrent extractions |
| `UN_DELETE_DELAY` | `5m` | Wait before cleaning extracted files |
| `UN_DELETE_ORIG` | `false` | Delete original archives after extraction |

> **Warning:** Don't set `UN_DELETE_ORIG=true` if you're seeding — it will delete the archive while you're still uploading.

## Verifying It Works

Check the logs:

```bash
docker logs unpackerr
```

You should see it connecting to each *arr on startup:

```
[INFO] Sonarr: http://sonarr:8989 - 0 queue items (connected)
[INFO] Radarr: http://radarr:7878 - 0 queue items (connected)
[INFO] Lidarr: http://lidarr:8686 - 0 queue items (connected)
```

## Troubleshooting

**"Connection refused" errors:** Make sure the *arr containers are running and the API keys are correct.

**Archives not extracting:** Check that the paths match — `UN_SONARR_0_PATHS_0` must point to where qBittorrent saves TV downloads inside the container (`/data/torrents/tv`).

**Extracted files not cleaned up:** This is controlled by `UN_DELETE_DELAY`. The *arr must successfully import the file before Unpackerr will clean up.

## Next Steps

- [Notifiarr](notifiarr.md) — notifications
- [Back to main README](../README.md)
