# MLPInfra

Docker Compose stack that runs the full MyLittlePlanet Minecraft network infrastructure.
A single `docker compose up -d` on a fresh VPS brings up everything. The rest is handled
by [MLPManager](https://github.com/mylittledevelop/MLPManager).

## What's Included

- **Pterodactyl Panel** — game server management panel (port 80)
- **Pterodactyl Wings** — daemon that runs game server containers on the host
- **MySQL 8** — database for the Panel (internal only)
- **Pterodactyl Redis** — queue/cache/session store for the Panel (internal only)
- **Minecraft Redis** — shared Redis instance for game server plugins (port 6400)
- **MLPManager** — fleet manager and bootstrap tool (port 8081)

## Prerequisites

- A VPS running Ubuntu 24.04 LTS
- Docker installed
- Ports 80, 6400, 8080, 8081, and your game server port range (25565–25600) open in your firewall

## Installation

### 1. Install Docker

```bash
curl -fsSL https://get.docker.com | sh
usermod -aG docker $USER
newgrp docker
```

### 2. Clone the repo

```bash
cd /opt
git clone https://github.com/mylittledevelop/MLPInfra.git
cd MLPInfra
```

### 3. Configure

```bash
cp .env.example .env
nano .env
```

Fill in the following values:

| Variable | Description |
|---|---|
| `SERVER_IP` | Your VPS public IP address |
| `TIMEZONE` | Your timezone e.g. `Europe/Prague` |
| `MYSQL_ROOT_PASSWORD` | Strong password for MySQL root |
| `MYSQL_PASSWORD` | Strong password for the Pterodactyl MySQL user |
| `APP_KEY` | Laravel app key — generate with `echo "base64:$(openssl rand -base64 32)"` |

### 4. Configure the Manager

```bash
cp manager-config.example.yml manager-config.yml
nano manager-config.yml
```

Fill in your panel URL, API keys, node specs, git credentials and server definitions.
See [MLPManager](https://github.com/mylittledevelop/MLPManager) for full config documentation.

### 5. Start everything

```bash
docker compose up -d
```

### 6. First time Panel setup

Run database migrations and create the admin user:

```bash
docker compose exec panel apk add mysql-client
docker compose exec panel php artisan migrate --force
docker compose exec panel php artisan p:user:make
```

### 7. Bootstrap your network

Open the manager in your browser:

```
http://YOUR_VPS_IP:8081
```

Hit **Bootstrap** — the manager will:
- Create the Pterodactyl location and node
- Generate and write the Wings configuration
- Restart Wings and wait for it to come online
- Create all port allocations
- Import your egg
- Create all game servers from your config

That's it. Your network is running.

## Updating

```bash
cd /opt/MLPInfra
git pull
docker compose up -d --build
```

## Services & Ports

| Service | Port | Description |
|---|---|---|
| Pterodactyl Panel | 80 | Web UI — admin and server management |
| Wings Daemon | 8080 | Panel ↔ Wings communication (internal) |
| Wings SFTP | 2022 | SFTP access to server files |
| MLPManager | 8081 | Fleet manager web UI |
| Minecraft Redis | 6400 | Shared Redis for game server plugins |
| Game Servers | 25565–25600 | Minecraft server ports |

## File Structure

```
MLPInfra/
├── docker-compose.yml       ← full stack definition
├── .env                     ← secrets, never committed
├── .env.example             ← template for .env
├── manager-config.yml       ← manager config, never committed
├── manager-config.example.yml ← template for manager config
└── start.sh                 ← safe startup script with cleanup on failure
```
