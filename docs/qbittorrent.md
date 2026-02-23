# qBittorrent

qBittorrent is the download client — it handles the actual torrent downloads. The *arrs send download requests here.

## Compose Configuration

```yaml
qbittorrent:
  image: lscr.io/linuxserver/qbittorrent:latest
  container_name: qbittorrent
  environment:
    - PUID=${PUID}
    - PGID=${PGID}
    - TZ=${TZ}
    - WEBUI_PORT=8080
  volumes:
    - ${CONFIG_DIR}/qbittorrent:/config
    - ${DATA_DIR}/torrents:/data/torrents
  ports:
    - "8080:8080"
    - "6881:6881"
    - "6881:6881/udp"
  restart: unless-stopped
```

Key points:

- **Port 8080** — Web UI access
- **Port 6881** — incoming torrent connections (forward this on your router for best speeds)
- Only the `torrents` subdirectory is mounted, not all of `DATA_DIR`

## First-Run Setup

1. Start the container: `docker compose up -d qbittorrent`
2. Check logs for the temporary password:

   ```bash
   docker logs qbittorrent
   ```

   Look for: `The WebUI administrator password was not set. A temporary password is provided...`

3. Open `http://your-server-ip:8080`
4. Log in with username `admin` and the temporary password
5. Go to **Tools → Options → Web UI** and set a proper password

## Configure Categories

Categories ensure downloads go to the correct subdirectory. The *arrs create categories automatically when connected, but you can set them up manually:

**Right-click in the left panel → Add category**

| Category | Save Path |
|---|---|
| `tv` | `/data/torrents/tv` |
| `movies` | `/data/torrents/movies` |
| `music` | `/data/torrents/music` |

## Recommended Settings

**Tools → Options:**

### Downloads

- **Default Save Path:** `/data/torrents`
- **Keep incomplete torrents in:** disabled (not needed with categories)

### Connection

- **Listening Port:** 6881
- Forward this port on your router for best download speeds
- **UPnP / NAT-PMP:** Enable if your router supports it

### Speed

- Set upload/download limits based on your connection. A common approach:
  - Upload limit: 80% of your upload bandwidth
  - Download limit: unlimited (or 90% of your download bandwidth)

### BitTorrent

- **Seeding Limits:** Set a ratio limit (e.g., 2.0) or time limit depending on your tracker rules
- **When ratio is reached:** Pause torrent (don't remove — let the *arrs handle removal)

## Connecting to the *arrs

Each *arr connects to qBittorrent as a download client:

| App | Category |
|---|---|
| Sonarr | `tv` |
| Radarr | `movies` |
| Lidarr | `music` |

In each *arr: **Settings → Download Clients → + → qBittorrent**

| Field | Value |
|---|---|
| Host | `qbittorrent` |
| Port | `8080` |
| Username | admin |
| Password | your password |
| Category | (see table above) |

## VPN Integration

If you want to route qBittorrent through a VPN, consider using a VPN container like `gluetun`:

```yaml
gluetun:
  image: qmcgaw/gluetun:latest
  container_name: gluetun
  cap_add:
    - NET_ADMIN
  environment:
    - VPN_SERVICE_PROVIDER=your_provider
    - VPN_TYPE=wireguard
    # ... provider-specific config
  ports:
    - "8080:8080"     # qBittorrent WebUI
    - "6881:6881"
    - "6881:6881/udp"

qbittorrent:
  # ... same as before, but:
  network_mode: "service:gluetun"
  # Remove the ports section (gluetun handles them)
```

This isn't included in the default compose file to keep things simple. Add it if needed.

## Troubleshooting

**Can't log in:** Check `docker logs qbittorrent` for the temporary password. If you've forgotten your password, delete `${CONFIG_DIR}/qbittorrent/qBittorrent/qBittorrent.conf` and restart.

**Slow speeds:** Make sure port 6881 is forwarded on your router. Check connection settings for any unnecessary limits.

**Downloads not going to the right folder:** Verify categories are set with correct save paths. The category save paths must match what the *arrs expect.

## Next Steps

- [Prowlarr](prowlarr.md) — set up indexers
- [Sonarr](sonarr.md) — connect for TV show automation
- [Back to main README](../README.md)
