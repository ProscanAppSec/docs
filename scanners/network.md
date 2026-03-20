# Network Scanner

The network scanner discovers hosts, maps open ports, fingerprints running services, detects operating systems, and matches findings against CVE databases.

## What It Does

- **Host discovery** — find active hosts on a network range
- **Port scanning** — TCP and UDP scanning across any port range
- **Service fingerprinting** — identify what's running on each open port using 7,000+ fingerprint patterns
- **OS detection** — determine the operating system through TTL analysis, TCP options, banners, and port profiles
- **CVE matching** — cross-reference detected services and versions against the NVD for known vulnerabilities

## Scan Modes

| Mode | Description |
|------|------------|
| Discovery | Quick host enumeration — just find what's on the network |
| Quick | Top ports scan with basic service detection |
| Full | All ports, full service fingerprinting, OS detection |
| Stealth | Slower, less detectable scan techniques |
| Custom | Choose your own port range, timing, and options |

## Security Scripts

The scanner includes 600+ built-in scripts for targeted checks:

- SSL/TLS configuration analysis
- Default credential testing
- SMB vulnerability detection
- DNS misconfigurations
- HTTP method enumeration
- Service-specific vulnerability checks

## CVE Matching

Detected service versions are automatically matched against the National Vulnerability Database. Results include CVSS scores and references to published advisories.

## Usage

1. Go to **Network Scanner**
2. Enter a target IP, hostname, or CIDR range
3. Choose a scan mode
4. Run the scan

Results show discovered hosts, open ports, identified services, and any CVE matches — organized by host.
