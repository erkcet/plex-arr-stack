# Networking & Remote Access

By default, all services are accessible only on your local network. This guide covers options for secure remote access.

## Local Access (Default)

After starting the stack, services are available at `http://your-server-ip:PORT`:

| Service | URL |
|---|---|
| Plex | `http://your-server-ip:32400/web` |
| Sonarr | `http://your-server-ip:8989` |
| Radarr | `http://your-server-ip:7878` |
| Lidarr | `http://your-server-ip:8686` |
| Prowlarr | `http://your-server-ip:9696` |
| qBittorrent | `http://your-server-ip:8080` |
| Autobrr | `http://your-server-ip:7474` |
| Bazarr | `http://your-server-ip:6767` |
| Tautulli | `http://your-server-ip:8181` |
| Notifiarr | `http://your-server-ip:5454` |
| Homepage | `http://your-server-ip:3000` |

## Option 1: Cloudflare Tunnel (Recommended)

Cloudflare Tunnels expose services to the internet without opening any ports on your router. Traffic is encrypted and routed through Cloudflare's network.

### Prerequisites

- A domain managed by Cloudflare (free tier works)
- A Cloudflare account

### Setup

1. Install cloudflared:

   ```bash
   # Add to your compose file or install on the host
   curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg
   echo "deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflared.list
   sudo apt update && sudo apt install cloudflared
   ```

2. Authenticate:

   ```bash
   cloudflared tunnel login
   ```

3. Create a tunnel:

   ```bash
   cloudflared tunnel create media-stack
   ```

4. Create config at `/etc/cloudflared/config.yml`:

   ```yaml
   tunnel: your-tunnel-id
   credentials-file: /etc/cloudflared/your-tunnel-id.json

   ingress:
     - hostname: plex.yourdomain.com
       service: http://localhost:32400
     - hostname: sonarr.yourdomain.com
       service: http://localhost:8989
     - hostname: radarr.yourdomain.com
       service: http://localhost:7878
     - hostname: dash.yourdomain.com
       service: http://localhost:3000
     - service: http_status:404
   ```

5. Route DNS:

   ```bash
   cloudflared tunnel route dns media-stack plex.yourdomain.com
   cloudflared tunnel route dns media-stack sonarr.yourdomain.com
   # ... repeat for each hostname
   ```

6. Run as a service:

   ```bash
   sudo cloudflared service install
   sudo systemctl enable --now cloudflared
   ```

### Cloudflare Access (Optional)

Add an extra authentication layer via Cloudflare Zero Trust:

1. Go to Cloudflare Zero Trust dashboard
2. Create an Access Application for your hostnames
3. Add an authentication policy (email OTP, Google SSO, etc.)

This protects your services even if someone guesses the URL.

## Option 2: Reverse Proxy (Nginx Proxy Manager)

If you prefer a traditional reverse proxy with Let's Encrypt SSL:

Add to your `docker-compose.yml`:

```yaml
nginx-proxy-manager:
  image: jc21/nginx-proxy-manager:latest
  container_name: npm
  ports:
    - "80:80"
    - "443:443"
    - "81:81"      # Admin UI
  volumes:
    - ${CONFIG_DIR}/npm/data:/data
    - ${CONFIG_DIR}/npm/letsencrypt:/etc/letsencrypt
  restart: unless-stopped
```

Then:

1. Open `http://your-server-ip:81`
2. Default login: `admin@example.com` / `changeme`
3. Add Proxy Hosts for each service
4. Enable SSL with Let's Encrypt (automatic)

**Requires:** Ports 80 and 443 forwarded on your router, and a domain with DNS pointed to your public IP.

## Option 3: Tailscale

Tailscale creates a private VPN between your devices — no ports to open, no domain needed. Great for personal access.

1. Install Tailscale on your server and devices: https://tailscale.com/download
2. Access services via Tailscale IP: `http://100.x.y.z:PORT`

Pros: zero configuration, very secure, works behind CGNAT.
Cons: every client needs Tailscale installed, not ideal for sharing with others.

## Plex Remote Access

Plex has its own remote access system:

1. **Settings → Remote Access** in Plex
2. Port forward TCP 32400 on your router
3. Plex handles encryption and auth internally

You can use Plex's built-in remote access alongside any of the above options for the *arr services.

If you can't port forward (CGNAT), use a Cloudflare tunnel or Tailscale.

## Security Checklist

- [ ] Every service has authentication enabled (username + password)
- [ ] API keys are not exposed publicly
- [ ] If using a reverse proxy, SSL/TLS is enabled
- [ ] Firewall is configured (only necessary ports open)
- [ ] Consider Cloudflare Access or similar for additional authentication
- [ ] qBittorrent WebUI is not directly exposed to the internet

## Next Steps

- [Maintenance](maintenance.md) — backups, updates, and troubleshooting
- [Back to main README](../README.md)
