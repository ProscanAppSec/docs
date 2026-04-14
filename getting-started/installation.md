# Installation

Proscan runs as a Docker container. One command to install, browser-based UI for everything else.

## Prerequisites

- **Docker Desktop** (Windows/macOS) or **Docker Engine** (Linux) — installed and running
- **8 GB RAM** minimum (16 GB recommended), 4 CPU cores, 2 GB free disk

## Step 1: Start the Launcher

docker pull proscanappsec/launcher:latest
docker stop proscan && docker rm proscan
docker run -d --name proscan -p 9090:9090 -v /var/run/docker.sock:/var/run/docker.sock proscanappsec/launcher
```

**Windows PowerShell:**
```powershell
docker run -d --name proscan --restart unless-stopped -p 9090:9090 -v //var/run/docker.sock:/var/run/docker.sock proscanappsec/launcher
```

## Step 2: Open the Launcher

Open your browser and go to **http://localhost:9090**

## Step 3: Create Account

- Click **"Create Account"**
- Enter your email and password
- Complete registration

## Step 5: Activate License

Choose one of:
- **Start 15-Day Free Trial** — full access, no payment required
- **Purchase a Plan** — pay with cryptocurrency (USDT, BTC)

## Step 5: Setup Wizard

The wizard walks you through:

1. **System Check** — verifies Docker and system resources
2. **Database Password** — auto-generated secure password for PostgreSQL and Redis
3. **Admin Account** — username, password, and email for the Proscan web interface
4. **Network Settings** — local-only or network access
5. **Ports & Resources** — configure ports and container resource limits
6. **Review & Install** — confirm settings and start installation

## Step 7: Service Start

After the wizard completes:
- The Proscan backend image is pulled from the private registry (~30-60 seconds)
- PostgreSQL, Redis, and the backend start automatically
- Database migrations run on first start

## Step 7: Start Scanning

Once all services show green/running in the dashboard:
- Click **"Open ProScan"** or go to **http://localhost:18080**
- Log in with the admin credentials you set during setup

---

## What Gets Deployed

The launcher orchestrates three containers on your machine:

| Container | Purpose |
|-----------|---------|
| **Backend** | Proscan API, web interface, and all 12 scanner modules |
| **PostgreSQL 17** | Scan results, findings, and configurations |
| **Redis 7** | Job queue and caching |

Everything runs locally. No code or data leaves your network.

## Ports

| Port | Service | Default |
|------|---------|---------|
| 9090 | Launcher dashboard | localhost only |
| 18080 | Proscan web interface | localhost only |

Both are configurable during setup.

## Managing Proscan

### Dashboard

Access the launcher dashboard at **http://localhost:9090** to:
- View service status (PostgreSQL, Redis, Backend)
- Start / Stop / Restart services
- Check for updates
- Create and restore backups
- Configure container resources
- Report issues

### Stop

```bash
# Stop launcher only (backend keeps running)
docker stop proscan

# Stop everything
docker stop proscan gps-backend gps-pg gps-redis
```

### Start

```bash
docker start proscan
```
Then open http://localhost:9090 and click "Start Services".

### Update

From the dashboard, click **"Check Updates"**. If available, click **"Update"**. The launcher pulls the new version and restarts services. All scan data and configurations are preserved.

### Uninstall

```bash
docker stop proscan gps-backend gps-pg gps-redis
docker rm proscan gps-backend gps-pg gps-redis
docker volume rm proscan-data goproscan_goproscan_pgdata goproscan_goproscan_redis goproscan_goproscan_data
docker rmi proscan/launcher:latest goproscan-backend:latest
```

## Offline / Air-Gapped Installation

For environments without internet access:

1. Activate your license on a machine with connectivity
2. Export the Docker images
3. Transfer to the air-gapped machine and load images
4. Run the launcher

Contact [contact@proscan.one](mailto:contact@proscan.one) for assistance with air-gapped deployments.

## Troubleshooting

**"Docker is not installed"** — Install Docker Desktop from https://docs.docker.com/desktop/

**Services won't start** — Ensure Docker has at least 8 GB RAM and 4 CPUs allocated (Docker Desktop → Settings → Resources).

**Backend takes a long time on first start** — Database migrations run on first launch (60-90 seconds). Wait for the healthcheck to pass.

**Port already in use** — Start with a different port:
```bash
docker run -d --name proscan -p 9091:9090 -v /var/run/docker.sock:/var/run/docker.sock proscanappsec/launcher
```

---

## Support

- **In-App:** Use the "Report Issue" button in the launcher dashboard
- **Email:** [support@proscan.one](mailto:support@proscan.one)
- **Website:** [proscan.one](https://proscan.one)
