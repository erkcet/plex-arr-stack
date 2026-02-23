# Plex

Plex is the media server — it organizes and streams your library to any device.

## Compose Configuration

```yaml
plex:
  image: lscr.io/linuxserver/plex:latest
  container_name: plex
  network_mode: host
  environment:
    - PUID=${PUID}
    - PGID=${PGID}
    - TZ=${TZ}
    - VERSION=docker
    - PLEX_CLAIM=${PLEX_CLAIM}
  volumes:
    - ${CONFIG_DIR}/plex:/config
    - ${DATA_DIR}/media:/data/media
  devices:
    - /dev/dri:/dev/dri
  restart: unless-stopped
```

Key points:

- **`network_mode: host`** — required for Plex to be discoverable on your local network (DLNA, GDM). Port 32400 is exposed directly.
- **`PLEX_CLAIM`** — ties the server to your Plex account on first run. Get a token at [plex.tv/claim](https://plex.tv/claim) (valid for 4 minutes).
- **`/dev/dri`** — passes the GPU for hardware transcoding. Remove this line if you don't have Intel/AMD integrated graphics.

## First-Run Setup

1. Start Plex: `docker compose up -d plex`
2. Open `http://your-server-ip:32400/web`
3. Sign in with your Plex account (the claim token links it automatically)
4. Give your server a name

## Adding Libraries

Add three libraries pointing to the container paths:

| Library Type | Folder |
|---|---|
| Movies | `/data/media/movies` |
| TV Shows | `/data/media/tv` |
| Music | `/data/media/music` |

Settings per library:

- **Scanner:** Plex Movie / Plex TV Series / Plex Music
- **Agent:** Plex Movie / Plex TV Series / Plex Music
- Enable **Generate video preview thumbnails** if you have the CPU/storage for it

## Remote Access

Go to **Settings → Remote Access**:

- **Manually specify public port:** 32400 (or your forwarded port)
- Your router needs to forward TCP 32400 to your server's LAN IP

If you're behind CGNAT or can't port forward, see [docs/networking.md](networking.md) for reverse proxy and Cloudflare tunnel options.

## Hardware Transcoding

Requires **Plex Pass**.

1. Verify your GPU is visible in the container:

   ```bash
   docker exec plex ls -la /dev/dri
   ```

2. In Plex: **Settings → Transcoder**:
   - Enable **Use hardware acceleration when available**
   - Enable **Use hardware-accelerated video encoding**

3. Test by playing something and forcing transcoding (set quality to a lower resolution in the Plex client).

### No GPU?

Remove the `devices` section from the compose file. Plex will fall back to software transcoding (CPU-heavy).

## Recommended Settings

- **Settings → Library:** Enable "Scan my library automatically" and "Run a partial scan when changes are detected"
- **Settings → Scheduled Tasks:** Set automatic scan to run daily
- **Settings → Network:** Set "Secure connections" to "Preferred"

## Connecting to Other Services

- **Tautulli** connects to Plex for monitoring — see [tautulli.md](tautulli.md)
- **Overseerr / Ombi** (not included in this stack) can connect for media requests

## Troubleshooting

**Plex not found on network:** Make sure `network_mode: host` is set. Bridge mode breaks local discovery.

**Claim token expired:** Remove the Plex config directory and restart with a fresh token:

```bash
docker compose down plex
rm -rf ${CONFIG_DIR}/plex
# Get new claim token, update .env
docker compose up -d plex
```

**Transcoding errors:** Check `/config/Library/Application Support/Plex Media Server/Logs/` inside the container for details.

## Next Steps

- [Bazarr](bazarr.md) — automatic subtitles for your Plex library
- [Tautulli](tautulli.md) — monitor who's watching what
- [Back to main README](../README.md)
