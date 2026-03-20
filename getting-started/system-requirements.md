# System Requirements

Proscan runs entirely on your infrastructure. Below are the minimum and recommended specs.

## Operating System

| OS | Minimum Version |
|----|----------------|
| Windows | 10 (64-bit) or Server 2019 |
| Linux | Ubuntu 20.04, Debian 11, RHEL 8, or equivalent |
| macOS | 13 Ventura (Apple Silicon) |

## Hardware

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 4 cores | 8 cores |
| RAM | 16 GB | 32 GB |
| Disk | 20 GB free | 50 GB+ free |

Disk usage grows over time as scan results accumulate. Plan accordingly if you're scanning large codebases or running frequent scans.

## Network

Proscan does not require a persistent internet connection after initial activation. It supports fully air-gapped deployments.

An outbound connection is needed only for:
- License activation (one-time)
- Downloading updates (when available)

All scanning and analysis happens locally. No source code, scan results, or telemetry data is transmitted externally.

## Browser

The web interface works in any modern browser:
- Chrome 90+
- Firefox 90+
- Edge 90+
- Safari 15+
