# Maintenance

Keeping your stack healthy — updates, backups, logs, and common troubleshooting.

## Updating Containers

All services use `latest` tags, so updating is straightforward:

```bash
cd /path/to/plex-arr-stack

# Pull latest images
docker compose pull

# Recreate containers with new images
docker compose up -d

# Clean up old images
docker image prune -f
```

### Update a single service

```bash
docker compose pull sonarr
docker compose up -d sonarr
```

### Checking current versions

```bash
docker compose ps
```

Or check each service's UI — most show their version in Settings → General or System → Status.

## Backups

### What to back up

| Priority | What | Path | Why |
|---|---|---|---|
| Critical | *arr databases | `${CONFIG_DIR}/sonarr/`, `radarr/`, etc. | Contains all your library metadata, history, settings |
| Important | Compose + env | `docker-compose.yml`, `.env` | Recreate your stack |
| Nice to have | Plex config | `${CONFIG_DIR}/plex/` | Watch history, playlists (can be large) |
| Skip | Media files | `${DATA_DIR}/media/` | Can be re-downloaded; back up separately if important |

### Built-in *arr backups

Sonarr, Radarr, Lidarr, and Prowlarr create automatic backups:

- **Location:** `${CONFIG_DIR}/<service>/Backups/`
- **Schedule:** Weekly (default)
- **Configure:** Settings → General → Backups in each *arr

### Manual backup script

```bash
#!/bin/bash
BACKUP_DIR="/opt/backups/plex-arr-stack"
CONFIG_DIR="/opt/plex-arr-stack/config"
DATE=$(date +%Y-%m-%d)

mkdir -p "$BACKUP_DIR"

# Stop services for consistent backup (optional but recommended)
# docker compose stop sonarr radarr lidarr prowlarr

# Backup configs
tar -czf "$BACKUP_DIR/configs-$DATE.tar.gz" \
  -C "$CONFIG_DIR" \
  sonarr radarr lidarr prowlarr qbittorrent bazarr tautulli autobrr

# Backup compose files
cp docker-compose.yml .env "$BACKUP_DIR/"

# Restart if stopped
# docker compose start sonarr radarr lidarr prowlarr

# Retain last 4 backups
ls -t "$BACKUP_DIR"/configs-*.tar.gz | tail -n +5 | xargs -r rm

echo "Backup complete: $BACKUP_DIR/configs-$DATE.tar.gz"
```

### Automate with cron

```bash
# Run weekly on Sunday at 3 AM
crontab -e
0 3 * * 0 /opt/plex-arr-stack/backup.sh
```

## Logs

### View container logs

```bash
# Follow logs in real-time
docker logs -f sonarr

# Last 100 lines
docker logs --tail 100 radarr

# All service logs at once
docker compose logs -f

# Specific service, last hour
docker logs --since 1h qbittorrent
```

### Log locations inside containers

Each *arr stores logs in its config directory:

- `${CONFIG_DIR}/sonarr/logs/`
- `${CONFIG_DIR}/radarr/logs/`
- etc.

### Useful log commands

```bash
# Check which containers are using the most disk
docker system df -v

# Check container resource usage
docker stats --no-stream
```

## Common Troubleshooting

### Container won't start

```bash
# Check logs for the error
docker logs <container_name>

# Check if the port is already in use
ss -tlnp | grep <port>

# Recreate the container
docker compose up -d <service>
```

### Permission errors

```bash
# Check ownership of data directories
ls -la ${DATA_DIR}/
ls -la ${CONFIG_DIR}/

# Fix permissions
sudo chown -R $(id -u):$(id -g) ${DATA_DIR}
sudo chown -R $(id -u):$(id -g) ${CONFIG_DIR}
```

### Disk space issues

```bash
# Check disk usage
df -h

# Find large files in data directory
du -sh ${DATA_DIR}/*

# Clean up Docker
docker system prune -a    # WARNING: removes all unused images
docker volume prune        # WARNING: removes unused volumes
```

### Service can't reach another service

```bash
# Test connectivity from one container to another
docker exec sonarr curl -s http://qbittorrent:8080

# Check if containers are on the same network
docker network inspect plex-arr-stack_default
```

> Remember: Plex uses `network_mode: host`, so other containers can't reach it by container name. Use your server's LAN IP instead.

### Database corruption (rare)

If an *arr's database becomes corrupted:

1. Stop the service: `docker compose stop sonarr`
2. Check for backups in `${CONFIG_DIR}/sonarr/Backups/`
3. Replace the database file with a backup
4. Restart: `docker compose start sonarr`

### Reset a service completely

```bash
docker compose stop <service>
rm -rf ${CONFIG_DIR}/<service>
docker compose up -d <service>
```

This removes all config and starts fresh. You'll need to reconfigure the service from scratch.

## Health Checks

Quick commands to verify everything is working:

```bash
# Are all containers running?
docker compose ps

# Check resource usage
docker stats --no-stream

# Test *arr API connectivity
curl -s "http://localhost:8989/api/v3/system/status?apikey=YOUR_KEY" | jq .version
curl -s "http://localhost:7878/api/v3/system/status?apikey=YOUR_KEY" | jq .version

# Check disk space
df -h ${DATA_DIR}
```

## Docker Compose Cheat Sheet

| Command | Description |
|---|---|
| `docker compose up -d` | Start all services |
| `docker compose down` | Stop and remove all containers |
| `docker compose stop` | Stop all (keep containers) |
| `docker compose restart` | Restart all |
| `docker compose pull` | Pull latest images |
| `docker compose logs -f` | Follow all logs |
| `docker compose ps` | Show running containers |
| `docker compose up -d <svc>` | Start/recreate one service |
| `docker compose exec <svc> bash` | Shell into a container |

## Next Steps

- [Back to main README](../README.md)
