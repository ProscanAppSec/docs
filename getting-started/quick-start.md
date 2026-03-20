# Quick Start

This guide gets you from installation to your first scan in under ten minutes.

## Prerequisites

- Proscan installed and running (see [Installation](installation.md))
- A source code repository or web application to scan

## Your First SAST Scan

1. Log in to the Proscan web interface
2. Navigate to **Projects** and create a new project
3. Enter your repository URL or upload a source archive
4. Select **SAST** as the scan type
5. Click **Start Scan**

Proscan analyzes your source code and reports findings with severity, location, and suggested fixes. Depending on the size of your codebase, the scan may take a few minutes.

## Your First DAST Scan

1. Go to **Scans** and click **New Scan**
2. Enter the URL of your running web application
3. Select a scan profile (start with **Quick** for a fast first run)
4. Click **Start Scan**

The scanner crawls your application, tests for common vulnerabilities, and reports results as they come in.

## Reviewing Results

Each finding includes:

- **Severity** — Critical, High, Medium, Low, Informational
- **Location** — file path and line number (SAST) or URL and parameter (DAST)
- **Description** — what the vulnerability is and why it matters
- **Fix suggestion** — the original code, the fixed version, and a diff showing the change

Findings can be triaged directly from the results view: confirm, mark as false positive, or assign for remediation.

## What to Scan Next

Once you're comfortable with basic scans, try:

- **SCA** to check your dependencies for known vulnerabilities
- **Secrets** to find leaked credentials in your codebase
- **IaC** to scan your Terraform, Kubernetes, or Docker configs
- **API Security** to test your REST or GraphQL endpoints

Each scanner is available from the scan creation screen. You can run multiple scanner types in a single scan or run them individually.
