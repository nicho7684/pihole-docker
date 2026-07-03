<div align="center"> <img src="assets/pi-hole"></div>
# Pi-hole Docker Deployment

<div align="center">

![Pi-hole](https://img.shields.io/badge/Pi--hole-Docker-00e5ff?style=for-the-badge&logo=pihole&logoColor=white&labelColor=0d0d0d)
![Docker Compose](https://img.shields.io/badge/Docker%20Compose-v2-a259ff?style=for-the-badge&logo=docker&logoColor=white&labelColor=0d0d0d)
![License](https://img.shields.io/badge/License-MIT-fcee0c?style=for-the-badge&labelColor=0d0d0d)

Network-wide ad blocking, DNS filtering, and a slick web dashboard — deployed with a single `docker compose up`.

</div>

---

##  Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Prerequisites](#-prerequisites)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Usage](#-usage)
- [Setting Pi-hole as Your DNS Server](#-setting-pi-hole-as-your-dns-server)
- [Updating](#-updating)
- [Backup & Restore](#-backup--restore)
- [Troubleshooting](#-troubleshooting)
- [Uninstalling](#-uninstalling)
- [License](#-license)

---

##  Overview

This repo spins up [Pi-hole](https://pi-hole.net/) as a Docker container using Docker Compose. Pi-hole acts as a DNS sinkhole that blocks ads and trackers network-wide — no per-device browser extensions needed.

---

##  Features

-  Network-wide ad and tracker blocking via DNS
-  Web dashboard for stats, query logs, and block-list management
-  Fully configurable via environment variables
-  Persistent storage for config and logs (survives container restarts/updates)
-  One-command deploy and update via Docker Compose

---

##  Prerequisites

- A Linux host (bare metal, VM, or homelab server) — Raspberry Pi works great
- [Docker](https://docs.docker.com/engine/install/) and [Docker Compose](https://docs.docker.com/compose/install/) installed
- Port `53` free on the host (stop `systemd-resolved` or any other local DNS resolver first — see [Troubleshooting](#-troubleshooting))
- A static local IP for the host (recommended, so every device can reliably point to it)

---

##  Installation

**1. Clone the repo**

```bash
git clone https://github.com/<your-username>/pihole-docker.git
cd pihole-docker
```

**2. Copy the example environment file**

```bash
cp .env.example .env
```

**3. Edit `.env`** with your timezone, host IP, and desired admin password (see [Configuration](#-configuration)).

**4. Launch the stack**

```bash
docker compose up -d
```

**5. Confirm it's running**

```bash
docker compose ps
```

Visit the admin dashboard at:

```
http://<host-ip>:8080/admin
```

---

##  Configuration

### `docker-compose.yml`

```yaml
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    hostname: pihole
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8080:80/tcp"
    environment:
      TZ: "${TZ}"
      FTLCONF_webserver_api_password: "${PIHOLE_PASSWORD}"
      FTLCONF_dns_listeningMode: "all"
    volumes:
      - ./etc-pihole:/etc/pihole
      - ./etc-dnsmasq.d:/etc/dnsmasq.d
    cap_add:
      - NET_ADMIN
    restart: unless-stopped
```

### `.env.example`

```bash
# Timezone (e.g. Africa/Dar_es_Salaam)
TZ=Africa/Dar_es_Salaam

# Admin dashboard password
PIHOLE_PASSWORD=changeme
```

| Variable          | Description                          | Default |
|-------------------|---------------------------------------|---------|
| `TZ`               | Timezone for logs and dashboard      | UTC     |
| `PIHOLE_PASSWORD`  | Web admin login password             | random  |

>  **Never commit your real `.env` file.** Keep only `.env.example` in version control — add `.env` to `.gitignore`.

---

##  Usage

| Task                        | Command                                      |
|------------------------------|-----------------------------------------------|
| Start the stack              | `docker compose up -d`                       |
| Stop the stack                | `docker compose down`                        |
| View logs                     | `docker compose logs -f pihole`              |
| Restart Pi-hole                | `docker compose restart pihole`             |
| Open a shell in the container | `docker exec -it pihole bash`               |
| Change admin password         | `docker exec -it pihole pihole setpassword` |

---

##  Setting Pi-hole as Your DNS Server

**Option A — Per device:** point each device's DNS settings to `<host-ip>`.

**Option B — Whole network (recommended):** log into your router's admin panel and set the primary DNS server to `<host-ip>`. Every device on the network is now filtered automatically.

>  Add a secondary/fallback DNS (like `1.1.1.1`) in your router settings so the network still resolves if the Pi-hole host goes down.

---

##  Updating

Pi-hole's Docker image handles its own internal updates. To pull the latest image:

```bash
docker compose pull
docker compose up -d
```

---

##  Backup & Restore

Pi-hole has a built-in Teleporter export for full config backups.

**Backup:**

```bash
docker exec pihole pihole-FTL --teleporter /etc/pihole/teleporter_backup.zip
```

**Restore:** upload the `.zip` via **Settings → Teleporter** in the web dashboard, or copy it back into `./etc-pihole` and restart the container.

Since `./etc-pihole` and `./etc-dnsmasq.d` are bind-mounted, you can also just back up those two folders directly.

---

##  Troubleshooting

**Port 53 already in use**
Something else on the host is listening on DNS (commonly `systemd-resolved`):

```bash
sudo systemctl disable --now systemd-resolved
```

Then edit `/etc/resolv.conf` to point to a public resolver temporarily, and restart the stack.

**Can't reach the dashboard**
- Check the container is running: `docker compose ps`
- Check the host firewall allows port `8080`
- Confirm you're using the host's actual LAN IP, not `localhost`, from other devices

**Devices aren't being filtered**
- Confirm the router or device DNS settings actually point to the Pi-hole host IP
- Flush the device's DNS cache
- Check **Query Log** in the dashboard to see if requests are arriving at all

---

##  Uninstalling

```bash
docker compose down -v
rm -rf etc-pihole etc-dnsmasq.d
```

Then revert your router/device DNS settings back to their original values.

---

##  License

Licensed under the [MIT License](LICENSE).
