# System Requirements

## Hardware

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| RAM | 8 GB | 16 GB |
| CPU | 4 cores | 8 cores |
| Disk | 20 GB free | 50 GB free |

## Software

| Component | Required |
|-----------|----------|
| Docker | Docker Desktop (Windows/macOS) or Docker Engine (Linux) |
| OS | Windows 10+ (x64), macOS 12+ (Intel or Apple Silicon), Linux (x64) |

## Docker Resource Allocation

If using Docker Desktop, ensure sufficient resources are allocated:

- Docker Desktop → Settings → Resources
- At least **8 GB RAM** and **4 CPUs** assigned to Docker

## Default Container Resources

| Container | Memory | CPUs | Purpose |
|-----------|--------|------|---------|
| Backend | 8 GB | 4.0 | API, web UI, scanner modules |
| PostgreSQL | 2 GB | 2.0 | Scan results, findings, configurations |
| Redis | 768 MB | 0.5 | Job queue, caching |

These can be adjusted in the launcher settings after installation.

## Network

- Internet access required for initial setup, license activation, and package download
- After setup, Proscan runs fully offline (within license period)
- Air-gapped deployments supported with offline license activation

## Ports

| Port | Service | Default Binding |
|------|---------|----------------|
| 9090 | Launcher UI (Docker option) | localhost only |
| 18080 | Proscan web interface | localhost only |
| 15432 | PostgreSQL (internal) | localhost only |
| 16379 | Redis (internal) | localhost only |

All ports are configurable during setup.
