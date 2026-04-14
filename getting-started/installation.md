# Installation

Proscan offers two ways to deploy — choose whichever fits your environment.

---

## Option A: Desktop Launcher (Recommended)

A native desktop application that manages everything. Download, run, sign in — the setup wizard handles the rest.

### Download

| Platform | Download | Size |
|----------|----------|------|
| **Windows x64** | [ProScan-windows-amd64.exe](https://github.com/ProscanAppSec/download/releases/latest/download/ProScan-windows-amd64.exe) | 16.3 MB |
| **Linux x64** | [ProScan-linux-amd64](https://github.com/ProscanAppSec/download/releases/latest/download/ProScan-linux-amd64) | 15.7 MB |
| **macOS Universal (DMG)** | [ProScan-1.0.0-macos-universal.dmg](https://github.com/ProscanAppSec/download/releases/latest/download/ProScan-1.0.0-macos-universal.dmg) | 23.7 MB |
| **macOS Universal (ZIP)** | [ProScan-1.0.0-macos-universal.zip](https://github.com/ProscanAppSec/download/releases/latest/download/ProScan-1.0.0-macos-universal.zip) | 15.6 MB |

All downloads available at [GitHub Releases](https://github.com/ProscanAppSec/download/releases/latest).

### Run

**Windows:** Double-click `ProScan-windows-amd64.exe`.

**Linux:**
```bash
chmod +x ProScan-linux-amd64
./ProScan-linux-amd64
```

**macOS (DMG):** Open the DMG, drag Proscan to Applications. On first launch, macOS may show a Gatekeeper warning. Bypass with:
```bash
xattr -d com.apple.quarantine /Applications/ProScan.app
```

**macOS (ZIP):** Extract and run. Same Gatekeeper bypass applies if needed.

The desktop launcher opens a native window with the full Proscan interface embedded — no browser needed. WebSocket connections provide real-time scan progress.

---

## Option B: Docker Launcher

A containerized launcher that runs in your browser. Ideal for headless servers, remote machines, or environments where you prefer browser-based access.

### Prerequisites

- Docker Desktop or Docker Engine installed and running
- Add the Proscan registry to Docker's insecure registries (one-time setup):

**Docker Desktop (Windows / macOS):**
1. Open Docker Desktop → Settings → Docker Engine
2. Add to the JSON config:
```json
{
  "insecure-registries": ["registry.proscan.one:5000"]
}
```
3. Click "Apply & Restart"

**Linux:**
```bash
sudo tee /etc/docker/daemon.json <<EOF
{
  "insecure-registries": ["registry.proscan.one:5000"]
}
EOF
sudo systemctl restart docker
```

### Start

```bash
docker run -d \
  --name proscan \
  --restart unless-stopped \
  -p 9090:9090 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  proscan/launcher:latest
```

**Windows PowerShell:**
```powershell
docker run -d --name proscan --restart unless-stopped -p 9090:9090 -v //var/run/docker.sock:/var/run/docker.sock proscan/launcher:latest
```

Then open **http://localhost:9090** in your browser.

---

## Setup (Both Options)

After launching, the flow is the same:

### 1. Create Account

- Click **"Create Account"**
- Enter your email and password
- Complete registration

### 2. Activate License

Choose one of:
- **Start 15-Day Free Trial** — full access, no payment required
- **Purchase a Plan** — pay with cryptocurrency (USDT, BTC)

### 3. Setup Wizard

The wizard walks you through:

1. **System Check** — verifies Docker and system resources
2. **Database Password** — auto-generated secure password for PostgreSQL and Redis
3. **Admin Account** — username, password, and email for the Proscan web interface
4. **Network Settings** — local-only or network access
5. **Ports & Resources** — configure ports and container resource limits
6. **Review & Install** — confirm settings and start installation

### 4. Service Start

After the wizard completes:
- The Proscan backend image is pulled and loaded (~30-60 seconds)
- PostgreSQL, Redis, and the backend start automatically
- Database migrations run on first start

### 5. Start Scanning

Once all services show green/running:
- **Desktop Launcher:** The web interface loads directly inside the launcher window
- **Docker Launcher:** Click **"Open ProScan"** or go to **http://localhost:18080**

Log in with the admin credentials you set during setup.

---

## System Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| RAM | 8 GB | 16 GB |
| CPU | 4 cores | 8 cores |
| Disk | 20 GB | 50 GB |
| Docker | Docker Desktop or Engine | Latest stable |
| OS | Windows 10+, macOS 12+, Linux (x64) | — |

## Ports

| Service | Port | Description |
|---------|------|-------------|
| Launcher UI (Docker option) | 9090 | Setup wizard, dashboard, service management |
| Proscan App | 18080 | Security scanner web interface |

Both ports bind to `127.0.0.1` (localhost only) by default. Network access can be enabled during setup.

## Offline / Air-Gapped Installation

For environments without internet access:

1. Activate your license on a machine with connectivity
2. Download the package file
3. Transfer to the air-gapped machine
4. Run the launcher and use offline activation with your license file

Contact [contact@proscan.one](mailto:contact@proscan.one) for assistance with air-gapped deployments.

## Managing Proscan

### Dashboard

Access the launcher dashboard to:
- View service status (PostgreSQL, Redis, Backend)
- Start / Stop / Restart services
- Check for updates
- Create and restore backups
- Configure container resources
- Report issues

### Stop Services

**Desktop Launcher:** Use the close dialog — choose "Keep Running" (containers stay up) or "Shutdown" (everything stops).

**Docker Launcher:**
```bash
# Stop launcher only (backend keeps running)
docker stop proscan

# Stop everything
docker stop proscan gps-backend gps-pg gps-redis
```

### Updates

From the dashboard, click **"Check Updates"**. If an update is available, click **"Update"**. The launcher pulls the new version and restarts services. All scan data, findings, and configurations are preserved.

### Backup & Restore

Built-in backup and restore through the dashboard. Backups are password-encrypted files containing your full database. Create regular backups before updates or migrations.

### Uninstall

**Desktop Launcher:** Delete the executable. To remove all data:
```bash
docker stop gps-backend gps-pg gps-redis
docker rm gps-backend gps-pg gps-redis
docker volume rm goproscan_pgdata goproscan_redis goproscan_data
```

**Docker Launcher:**
```bash
docker stop proscan gps-backend gps-pg gps-redis
docker rm proscan gps-backend gps-pg gps-redis
docker volume rm proscan-data goproscan_pgdata goproscan_redis goproscan_data
docker rmi proscan/launcher:latest goproscan-backend:latest
```

## Troubleshooting

### "Docker is not installed"
Install Docker Desktop from https://docs.docker.com/desktop/

### Services won't start
Check Docker has enough resources allocated: Docker Desktop → Settings → Resources → at least 8 GB RAM, 4 CPUs.

### Backend takes a long time on first start
The backend runs database migrations on first launch, which can take 60-90 seconds. Wait for the healthcheck to pass.

### Port already in use
Change the port in the launcher settings, or for the Docker option:
```bash
docker run -d --name proscan -p 9091:9090 -v /var/run/docker.sock:/var/run/docker.sock proscan/launcher:latest
```

---

## Support

- **In-App:** Use the "Report Issue" button in the launcher dashboard
- **Email:** [support@proscan.one](mailto:support@proscan.one)
- **Website:** [proscan.one](https://proscan.one)
