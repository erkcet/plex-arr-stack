# Prerequisites

## Hardware Recommendations

### Minimum

- **CPU:** 2 cores (4 if Plex transcoding)
- **RAM:** 4 GB (8 GB recommended)
- **Storage:** SSD for configs, HDD(s) for media
- **Network:** Gigabit Ethernet recommended

### Plex Transcoding

If you plan to transcode (convert video on-the-fly for remote streaming):

- **CPU:** Intel Quick Sync (7th gen+) or AMD with VAAPI for hardware transcoding
- **Plex Pass:** Required for hardware transcoding
- **Without hardware transcoding:** ~2000 PassMark score per 1080p stream

If all your clients direct-play, transcoding hardware doesn't matter.

## OS Setup

Any Linux distro with Docker support works. Common choices:

- **Ubuntu Server 22.04 / 24.04** — most guides target this
- **Debian 12** — stable, lightweight
- **Proxmox** — if running in a VM/LXC

Make sure your system is up to date:

```bash
sudo apt update && sudo apt upgrade -y
```

## Install Docker

### Docker Engine (recommended)

Follow the official docs for your distro: https://docs.docker.com/engine/install/

Quick version for Ubuntu/Debian:

```bash
# Remove old versions
sudo apt remove docker docker-engine docker.io containerd runc 2>/dev/null

# Install prerequisites
sudo apt install -y ca-certificates curl gnupg

# Add Docker's GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Post-install: run Docker without sudo

```bash
sudo usermod -aG docker $USER
# Log out and back in for group changes to take effect
```

### Verify installation

```bash
docker --version
docker compose version
```

## User and Group IDs

All LinuxServer.io containers use `PUID` and `PGID` environment variables to run as your user, avoiding permission issues.

Find your IDs:

```bash
id
# uid=1000(youruser) gid=1000(youruser) ...
```

Use these values for `PUID` and `PGID` in your `.env` file.

## Next Steps

- [Storage setup](storage.md) — create the directory structure
- [Back to main README](../README.md)
