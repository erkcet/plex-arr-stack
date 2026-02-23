# Notifiarr

Notifiarr is a notification relay that sends alerts from Sonarr, Radarr, Lidarr, Plex, and other services to Discord, Telegram, Slack, and more — with rich formatting and media artwork.

## Compose Configuration

```yaml
notifiarr:
  image: golift/notifiarr:latest
  container_name: notifiarr
  hostname: notifiarr
  environment:
    - DN_API_KEY=${NOTIFIARR_API_KEY}
    - TZ=${TZ}
  volumes:
    - ${CONFIG_DIR}/notifiarr:/config
    - /var/run/utmp:/var/run/utmp
  ports:
    - "5454:5454"
  restart: unless-stopped
```

## Prerequisites

1. Create a free account at [notifiarr.com](https://notifiarr.com)
2. Get your API key from the site dashboard
3. Add it to `.env`:

   ```
   NOTIFIARR_API_KEY=your_api_key_here
   ```

## First-Run Setup

1. Open `http://your-server-ip:5454`
2. The local client shows connection status
3. Most configuration happens on the [Notifiarr website](https://notifiarr.com), not the local UI

## Website Configuration

Log in at [notifiarr.com](https://notifiarr.com) and configure:

### 1. Notification Targets

**Configuration → Notifications**

Add your notification channels:

- **Discord** — webhook URL or bot token
- **Telegram** — bot token and chat ID
- **Slack** — webhook URL
- **Pushover** — user key and API token

### 2. Connect *arr Apps

**Configuration → Integrations**

Add each *arr service:

| App | URL | API Key |
|---|---|---|
| Sonarr | `http://your-server-ip:8989` | Sonarr API key |
| Radarr | `http://your-server-ip:7878` | Radarr API key |
| Lidarr | `http://your-server-ip:8686` | Lidarr API key |

> Use your server's actual IP, not container names — the Notifiarr website connects from outside Docker.

### 3. Configure Triggers

Choose what events to notify on:

- **Sonarr/Radarr/Lidarr:** Grab, download, upgrade, health check
- **Plex:** Play start/stop, new content, server status
- **System:** Disk space, service health

## Webhook Setup in *arrs

Each *arr needs a Notifiarr webhook:

In Sonarr/Radarr/Lidarr: **Settings → Connect → + → Webhook**

| Field | Value |
|---|---|
| Name | Notifiarr |
| URL | `http://notifiarr:5454/api/v1/notification/sonarr` |
| Method | POST |

Replace `sonarr` in the URL with the appropriate app name (`radarr`, `lidarr`).

## What It Looks Like

Notifiarr sends formatted notifications with:

- Media poster/artwork
- Quality and size info
- Download status (grabbed, imported, upgraded)
- Links to the media in your *arr

This is much richer than the built-in notification options in each *arr.

## Notifiarr vs Built-in Notifications

| Feature | Built-in (*arr) | Notifiarr |
|---|---|---|
| Rich media embeds | No | Yes |
| Cross-app dashboard | No | Yes |
| Centralized config | No | Yes |
| Custom formatting | Limited | Extensive |
| Free tier | N/A | Yes (with limits) |

## Troubleshooting

**Client not connecting:** Check that `DN_API_KEY` is set correctly. View logs: `docker logs notifiarr`.

**Notifications not arriving:** Check the Notifiarr website dashboard for errors. Verify your notification targets (Discord webhook, Telegram bot) are working independently.

**Webhook errors in *arrs:** Make sure the webhook URL is correct and Notifiarr is reachable from the *arr container. Test with: `docker exec sonarr curl -s http://notifiarr:5454`.

## Next Steps

- [Homepage](homepage.md) — dashboard for all services
- [Back to main README](../README.md)
