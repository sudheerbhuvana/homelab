# Media (Servarr Stack)

Self-hosted media automation stack running on ZimaOS (`10.0.0.10`) using Docker Compose, with torrent and indexer traffic routed through a Gluetun WireGuard tunnel (`oci-amd-1`).

## Architecture & Services

All containers run on the `servarrnetwork` bridge (`172.30.0.0/24`) with `PUID=999`, `PGID=1000`, `TZ=Asia/Kolkata`, and `/DATA/data` mounted to `/data` for atomic moves and hardlinks.

| Service | Network / IP | Port | Purpose |
|---|---|---|---|
| `gluetun` | `172.30.0.2` | `8080`, `6881`, `9696` | Custom WireGuard VPN gateway (`oci-amd-1`) |
| `qbittorrent` | `service:gluetun` | `8080` (via `gluetun`) | Torrent client (VPN-routed) |
| `prowlarr` | `service:gluetun` | `9696` (via `gluetun`) | Indexer manager (VPN-routed) |
| `deunhealth` | — | — | Auto-restarts unhealthy containers |
| `sonarr` | `172.30.0.3` | `8989` | TV show management |
| `radarr` | `172.30.0.4` | `7878` | Movie management |
| `lidarr` | `172.30.0.5` | `8686` | Music management |
| `bazarr` | `172.30.0.6` | `6767` | Subtitle management |
| `seerr` | `172.30.0.7` | `5055` | Media request portal |

*(Optional `jellystat` and `jellystat-db` blocks are included in `compose.yaml` as comments.)*

---

## 1. ZimaOS Directory & Permission Setup

Create the app config directory and the unified `/DATA/data` media/torrents structure, then set ownership to `sudheer` (`999:1000`):

```bash
sudo mkdir -p /DATA/AppData/media/seerr \
  /DATA/data/torrents/{completed,incomplete,movies,tv,music} \
  /DATA/data/media/{movies,tv,music}

sudo chown -R 999:1000 /DATA/AppData/media /DATA/data
```

`ghcr.io/seerr-team/seerr` runs internally as UID/GID `1000:1000` (`node`) and ignores `PUID`/`PGID`, so `./seerr` must be owned by `1000:1000`:

```bash
sudo chown -R 1000:1000 /DATA/AppData/media/seerr
```

---

## 2. Environment Configuration (`.env`)

Copy `.env.example` to `.env` (never commit `.env`; it is ignored by `.gitignore`) and fill in your WireGuard credentials from `oci-amd-1` (`wg-easy`):

- `VPN_SERVICE_PROVIDER=custom` *(must be `custom` for self-hosted WireGuard)*
- `VPN_TYPE=wireguard`
- `WIREGUARD_ENDPOINT_IP`, `WIREGUARD_ENDPOINT_PORT=51820`
- `WIREGUARD_PUBLIC_KEY`, `WIREGUARD_PRIVATE_KEY`, `WIREGUARD_PRESHARED_KEY`, `WIREGUARD_ADDRESSES`
- `PUID=999`, `PGID=1000`, `TZ=Asia/Kolkata`, `DATA_DIR=/DATA/data`

---

## 3. Deploy to ZimaOS

1. Copy `compose.yaml` and `.env` from your workstation to ZimaOS:
   ```powershell
   scp d:\Homelab\media\compose.yaml d:\Homelab\media\.env sudheer@10.0.0.10:/DATA/AppData/media/
   ```
2. Stop and remove any standalone containers that share ports (keep `jellyfin` and `arcane-edge-agent` untouched):
   ```bash
   sudo docker stop qbittorrent prowlarr sonarr radarr && sudo docker rm qbittorrent prowlarr sonarr radarr
   ```
3. Start the stack:
   ```bash
   cd /DATA/AppData/media
   sudo docker compose up -d
   ```

> **Note:** Whenever `gluetun` is recreated (`--force-recreate gluetun`), also recreate `qbittorrent` and `prowlarr` because they share its network namespace (`network_mode: service:gluetun`):
> ```bash
> sudo docker compose up -d --force-recreate gluetun qbittorrent prowlarr
> ```

---

## 4. Verify VPN & Containers

Check container status and confirm `gluetun` exits through `oci-amd-1`:

```bash
sudo docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
sudo docker exec gluetun sh -c 'wget -O- https://ifconfig.me; echo'
```

---

## 5. App Configuration Reference

### qBittorrent (`http://10.0.0.10:8080`)

1. **Fix LAN `Unauthorized` error** (caused by Host Header Validation when accessed via `10.0.0.10:8080`):
   ```bash
   cd /DATA/AppData/media
   sudo docker compose stop qbittorrent
   CONF=$(sudo find ./qbittorrent -name "qBittorrent.conf" | head -n 1)
   sudo sed -i '/^WebUI\\HostHeaderValidation=/d' "$CONF"
   sudo sed -i '/^\[Preferences\]/a WebUI\\HostHeaderValidation=false' "$CONF"
   sudo docker compose start qbittorrent
   ```
2. **Get the initial temporary password:**
   ```bash
   sudo docker logs qbittorrent | grep -i password
   ```
   Log in as `admin` and set a permanent password under **Tools → Options → Web UI**.
3. **Download paths (Tools → Options → Downloads):**
   - **Default Torrent Management Mode:** `Automatic`
   - **Default Save Path:** `/data/torrents/completed`
   - **Keep incomplete torrents in:** Checked → `/data/torrents/incomplete`

### Root Folders (`*arr` apps)

- **Sonarr** (`http://10.0.0.10:8989`): `/data/media/tv`
- **Radarr** (`http://10.0.0.10:7878`): `/data/media/movies`
- **Lidarr** (`http://10.0.0.10:8686`): `/data/media/music`
- **Bazarr** (`http://10.0.0.10:6767`)
- **Seerr** (`http://10.0.0.10:5055`)
- **Prowlarr** (`http://10.0.0.10:9696`)

### Internal Container Hostnames (for linking apps)

- **qBittorrent in Sonarr / Radarr / Lidarr:** Host `gluetun`, Port `8080`
- **Prowlarr App Sync:**
  - Prowlarr Server: `http://gluetun:9696`
  - Sonarr Server: `http://sonarr:8989`
  - Radarr Server: `http://radarr:7878`
  - Lidarr Server: `http://lidarr:8686`

---

## Optional: VPN Port Forwarding (Later)

qBittorrent downloads and seeds without port forwarding. To become connectable for incoming peer connections through `oci-amd-1`:

1. Forward a TCP+UDP port (e.g., `6881`) on `oci-amd-1` (OCI Security List/NSG + `iptables`/`nftables`) to `10.8.0.2:6881` over `wg0`.
2. Uncomment `FIREWALL_VPN_INPUT_PORTS=6881` in `.env` and recreate `gluetun`, `qbittorrent`, and `prowlarr`.
3. In qBittorrent (**Tools → Options → Connection**), set **Port used for incoming connections** to `6881` and uncheck UPnP / NAT-PMP.
