# Torrents Docker Stack

[![Compose Validate](https://github.com/LBates2000/torrents-stack/actions/workflows/compose-validate.yml/badge.svg)](https://github.com/LBates2000/torrents-stack/actions/workflows/compose-validate.yml)
[![Backup Restore Reminder](https://github.com/LBates2000/torrents-stack/actions/workflows/backup-restore-reminder.yml/badge.svg)](https://github.com/LBates2000/torrents-stack/actions/workflows/backup-restore-reminder.yml)

This stack routes only qBittorrent through WireGuard.

## How it works
- `wireguard` is the VPN gateway.
- `qbittorrent` uses `network_mode: "service:wireguard"`, so torrent traffic goes through WireGuard.
- `jackett` and `flaresolverr` run on `app_net` and expose their own ports.
- Host access is published by the compose port mappings.

## Access
- qBittorrent Web UI: http://localhost:8090
- Jackett UI: http://localhost:9117
- FlareSolverr API: http://localhost:8191

## First-time setup
- Install Docker Desktop (or Docker Engine + Compose plugin).
- Place your WireGuard client config in `./configs/wireguard/` (for example `wg0.conf`).
- Copy `.env.example` to `.env` and adjust values for your host/network.
- Start the stack with `docker compose up -d`.

### Quick start
#### Bash
```bash
cp .env.example .env
docker compose up -d
```

#### PowerShell
```powershell
Copy-Item .env.example .env
docker compose up -d
```

## Commands
- Start: `docker compose up -d`
- Stop: `docker compose down`
- Check: `docker compose ps`

## Rebuild options
Full rebuild with fresh images:
```bash
docker compose down
docker compose pull
docker compose build --no-cache
docker compose up -d --remove-orphans
```

Force recreate containers:
```bash
docker compose down
docker compose build --no-cache
docker compose up --force-recreate -d
```

Full reset (remove volumes):
```bash
docker compose down -v
docker compose build --no-cache
docker compose up --force-recreate -d
```

## Config directories
- Local data is mounted from `./configs`.
- Torrent downloads are stored in `./downloads`.

## Image pinning
- Images are pinned by default in `docker-compose.yml` via environment-backed tags.
- Override image versions in `.env` using `WIREGUARD_IMAGE_TAG`, `FLARESOLVERR_IMAGE_TAG`, `JACKETT_IMAGE_TAG`, and `QBITTORRENT_IMAGE_TAG`.

## Startup order
- `wireguard` and `flaresolverr` start independently (no dependencies).
- `jackett` depends on `flaresolverr`.
- `qbittorrent` depends on `wireguard`, `jackett`, and `flaresolverr`.

## Healthchecks (service-specific)
- `wireguard`: checks that `wg0` exists, the interface is up, and a public IP is returned from `${WIREGUARD_IP_CHECK_URL:-https://api.ipify.org}`
- `flaresolverr`: checks `http://localhost:8191`
- `jackett`: checks `http://localhost:9117/` and accepts `200`, `301`, or `302`
- `qbittorrent`: checks `http://localhost:8090/` and accepts `200` or `302`

## Startup timing (observed)
- Typical clean startup on this host: ~70-80 seconds.
- Recent 3-run sample on current config:

| Metric | Seconds |
| --- | ---: |
| Min | 71.29 |
| Avg | 73.74 |
| Max | 77.90 |

- Shared healthcheck cadence: `interval=20s`, `timeout=10s`, `retries=8`, `start_period=30s`.
- If startup exceeds 2-3 minutes, check logs.

### Quick diagnostics
- Overall status: `docker compose ps`
- Health-focused status: `docker compose ps --format "table {{.Name}}\t{{.State}}\t{{.Health}}\t{{.Status}}"`
- Timed startup (PowerShell): `$t = Measure-Command { docker compose up -d }; $t.TotalSeconds`
- Tail all logs: `docker compose logs -f`
- Tail one service: `docker compose logs -f wireguard` (or `flaresolverr`, `jackett`, `qbittorrent`)

## Backup and restore
The repository includes a monthly reminder workflow that opens a backup/restore verification issue:
- `Backup Restore Reminder` workflow: https://github.com/LBates2000/torrents-stack/actions/workflows/backup-restore-reminder.yml

Backup runtime config (PowerShell):
```powershell
$stamp = Get-Date -Format "yyyyMMdd-HHmmss"
New-Item -ItemType Directory -Path .\backups -Force | Out-Null
Compress-Archive -Path .\configs\* -DestinationPath (".\\backups\\configs-" + $stamp + ".zip") -Force
```

Restore runtime config (PowerShell):
```powershell
docker compose down
Expand-Archive -Path .\backups\configs-<timestamp>.zip -DestinationPath .\configs -Force
docker compose up -d
```

After restore, verify health:
```powershell
docker compose ps --format "table {{.Name}}\t{{.State}}\t{{.Health}}\t{{.Status}}"
```

## Notes
- If you change WireGuard peer config, restart the `wireguard` container.
- Keep `wireguard` service healthy before app services start.
- Redirect-based healthchecks are intentional: `301`/`302` can still mean the web UI is up before login.

## Project governance
- Security policy: see `SECURITY.md`.
- Use GitHub issue templates for monthly maintenance and backup restore drills.

## Author
- Lawrence Bates (<Lawrence.Bates@gmail.com>)
