# Container Security

The container scanner analyzes container images for vulnerabilities, Dockerfile issues, and runtime configuration problems. It works with images from any standard container registry.

## What It Does

- **Vulnerability scanning** — checks every package in the image against CVE databases
- **Dockerfile analysis** — lints your Dockerfile for security anti-patterns (secrets in build args, running as root, using `latest` tags, bloated base images, end-of-life base images)
- **Layer analysis** — examines each image layer to identify where vulnerabilities were introduced
- **Runtime configuration** — checks for privileged containers, excessive capabilities, missing security profiles

## Supported Registries

Pull images from any registry that supports the Docker Registry v2 API:

- Docker Hub
- Amazon ECR
- Google Container Registry (GCR)
- GitHub Container Registry (GHCR)
- Azure Container Registry (ACR)
- Harbor
- JFrog Artifactory
- Self-hosted registries

Authentication credentials for private registries can be configured in the scan settings.

## SBOM Generation

Container scans produce a Software Bill of Materials listing every package in the image. Output formats include CycloneDX and SPDX.

## CIS Benchmarks

Container configurations are evaluated against the CIS Docker Benchmark, covering areas like image provenance, resource limits, network segmentation, and logging.

## Usage

1. Create a scan and select **Container**
2. Enter the image reference (e.g., `registry.example.com/app:v1.2.3`)
3. Configure registry credentials if needed
4. Run the scan

Results are organized by layer, so you can see exactly which base image or build step introduced each vulnerability.
