# Tautulli

Tautulli monitors your Plex server — tracking who's watching what, play history, statistics, and notifications.

## Compose Configuration

```yaml
tautulli:
  image: lscr.io/linuxserver/tautulli:latest
  container_name: tautulli
  environment:
    - PUID=${PUID}
    - PGID=${PGID}
    - TZ=${TZ}
  volumes:
    - ${CONFIG_DIR}/tautulli:/config
  ports:
    - "8181:8181"
  restart: unless-stopped
```

## First-Run Setup

1. Open `http://your-server-ip:8181`
2. The setup wizard starts automatically
3. **Connect to Plex:**
   - Sign in with your Plex account, or
   - Manual: Host = `your-server-ip`, Port = `32400`
4. Tautulli will verify the connection and begin monitoring

> Since Plex uses `network_mode: host`, Tautulli connects using your server's actual IP, not a Docker container name.

## What Tautulli Tracks

- **Current activity** — who's streaming, what they're playing, transcoding or direct play
- **History** — full play history with timestamps, duration, and user
- **Statistics** — most watched content, most active users, play counts
- **Libraries** — breakdown of your library by type, resolution, codec
- **Graphs** — play count trends, concurrent streams, bandwidth usage

## Notifications

**Settings → Notification Agents**

Tautulli can send alerts for various events:

- Playback started / stopped
- New content added to Plex
- Plex server down / back up
- Buffer warnings
- Concurrent stream limits

Supported notification services:

- Discord
- Telegram
- Slack
- Email
- Pushover
- Many more

### Example: Discord webhook

1. Create a webhook in your Discord server
2. In Tautulli: **Settings → Notification Agents → Add → Discord**
3. Paste the webhook URL
4. Select which triggers to notify on
5. Customize the notification text/embeds

## Newsletters

Tautulli can send periodic newsletters summarizing recently added content:

**Settings → Newsletters**

- Schedule: weekly or monthly
- Recipients: email addresses
- Content: recently added movies, TV episodes, music

## Recommended Settings

**Settings → General:**

- **Time Format:** 24-hour or 12-hour per your preference
- **Grouping:** Group play history by user (cleaner view)

**Settings → Homepage:**

- Customize which stats appear on the dashboard

## Troubleshooting

**Can't connect to Plex:** Make sure Plex is running and accessible. Since Plex uses host networking, use your server's LAN IP (not `plex` container name) when configuring the connection.

**History not recording:** Check that Tautulli is properly connected (Settings → Plex Media Server → Verify Server). Activity logging may take a moment to start after initial setup.

## Next Steps

- [Notifiarr](notifiarr.md) — centralized notifications for all services
- [Back to main README](../README.md)
