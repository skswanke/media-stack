# Media Stack

A self-hosted media request and download stack for UGREEN NAS (UGOS), routing all torrent traffic through a Mullvad VPN.

## Services

| Service | Purpose | Port |
|---|---|---|
| [Gluetun](https://github.com/qdm12/gluetun) | VPN gateway (Mullvad/WireGuard) with kill switch | — |
| [Seerr](https://github.com/seerr-team/seerr) | Media request manager | 5055 |
| [Radarr](https://radarr.video) | Movie collection manager | 7878 |
| [Sonarr](https://sonarr.tv) | TV show collection manager | 8989 |
| [Prowlarr](https://github.com/Prowlarr/Prowlarr) | Indexer manager for Radarr/Sonarr | 9696 |
| [qBittorrent](https://www.qbittorrent.org) | Torrent client (tunneled through VPN) | 8080 |
| [FlareSolverr](https://github.com/FlareSolverr/FlareSolverr) | Cloudflare bypass for Prowlarr indexers | 8191 |

qBittorrent runs on Gluetun's network stack — if the VPN drops, the kill switch blocks all torrent traffic automatically.

## Setup

### 1. VPN credentials

1. Go to [mullvad.net/en/account/wireguard-config](https://mullvad.net/en/account/wireguard-config) and generate a WireGuard config
2. Open the downloaded `.conf` file and copy:
   - `PrivateKey` → `WIREGUARD_PRIVATE_KEY` in `docker-compose.yaml`
   - `Address` (IPv4, e.g. `10.x.x.x/32`) → `WIREGUARD_ADDRESSES`
3. Set `SERVER_CITIES` to your preferred exit location (e.g. `Amsterdam`, `London`, `Zurich`)

### 2. Create directories on your NAS

```bash
# Config directories
mkdir -p /volume1/docker/{gluetun,seerr,radarr,sonarr,prowlarr,qbittorrent}

# Media directories
mkdir -p /volume1/media/{Downloads,Movies,TV}
```

### 3. Set timezone and user IDs

- Update `TZ` in the compose file to match your timezone (e.g. `America/Chicago`)
- Verify `PUID`/`PGID` match your NAS user — run `id` over SSH to check

### 4. Deploy (UGOS)

Docker App → Projects → New Project → select folder → paste the YAML → Deploy

Or via CLI:

```bash
docker compose up -d
```

## Web UIs

Replace `<NAS_IP>` with your NAS's local IP address:

- Seerr: `http://<NAS_IP>:5055`
- Radarr: `http://<NAS_IP>:7878`
- Sonarr: `http://<NAS_IP>:8989`
- Prowlarr: `http://<NAS_IP>:9696`
- qBittorrent: `http://<NAS_IP>:8080`

> **Note:** qBittorrent generates a temporary password on first launch. Check the Gluetun container logs to find it.

## Verify VPN is working

```bash
docker exec gluetun wget -qO- https://am.i.mullvad.net/connected
```

The response should confirm you're connected via Mullvad.
