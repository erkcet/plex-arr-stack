# Bazarr

Bazarr automatically downloads subtitles for your movies and TV shows, integrating directly with Sonarr and Radarr.

## Compose Configuration

```yaml
bazarr:
  image: lscr.io/linuxserver/bazarr:latest
  container_name: bazarr
  environment:
    - PUID=${PUID}
    - PGID=${PGID}
    - TZ=${TZ}
  volumes:
    - ${CONFIG_DIR}/bazarr:/config
    - ${DATA_DIR}/media:/data/media
  ports:
    - "6767:6767"
  restart: unless-stopped
```

Bazarr only needs access to the media directory (not torrents) since it adds subtitle files alongside your existing media.

## First-Run Setup

1. Open `http://your-server-ip:6767`
2. The setup wizard will guide you through initial configuration

## Connect to Sonarr

**Settings → Sonarr**

| Field | Value |
|---|---|
| Enabled | Yes |
| Host | `sonarr` |
| Port | `8989` |
| API Key | Sonarr's API key |

Click "Test" then "Save".

## Connect to Radarr

**Settings → Radarr**

| Field | Value |
|---|---|
| Enabled | Yes |
| Host | `radarr` |
| Port | `7878` |
| API Key | Radarr's API key |

Click "Test" then "Save".

## Configure Subtitle Providers

**Settings → Providers**

Add your subtitle sources. Popular free providers:

- **OpenSubtitles.com** — largest database, requires a free account
- **Subscene** — good for non-English content
- **Addic7ed** — community-driven, good quality

Some providers have API limits on free tiers. OpenSubtitles.com allows 20 downloads/day for free users.

## Set Languages

**Settings → Languages**

- Add your preferred subtitle languages
- Set the default language profile (applied to all series/movies unless overridden)
- Enable "Use Embedded Subtitles" if you want Bazarr to check for built-in subs before downloading

## Recommended Settings

**Settings → Subtitles:**

- **Subtitle Folder:** `Alongside Media File` (recommended — Plex picks them up automatically)
- **Anti-Captcha:** Not needed for most providers, but helps with OpenSubtitles if you hit limits
- **Performance:** Set search frequency based on your provider limits (default is fine)

**Settings → Scheduler:**

- Configure how often Bazarr checks for missing subtitles
- Default intervals are reasonable for most setups

## Troubleshooting

**No subtitles found:** Check your providers are configured and working (Settings → Providers → test each one). Some content, especially newer releases, may not have subtitles available yet.

**Wrong subtitles:** Bazarr scores subtitles by matching release info. If you get mismatched subs, try adding more providers or adjusting the minimum score in Settings → Subtitles.

**Subtitles not showing in Plex:** Make sure the subtitle file is in the same folder as the video with a matching name. Refresh metadata in Plex for the affected item.

## Next Steps

- [Tautulli](tautulli.md) — monitoring
- [Back to main README](../README.md)
