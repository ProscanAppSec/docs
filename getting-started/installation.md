# Installation

Proscan is distributed as a single executable. There is no complex installer — download it, run it, and follow the setup wizard.

## Step 1: Download

Log in to your account at [proscan.one](https://proscan.one) and download the Proscan package for your operating system:

- `proscan` — Windows (64-bit)
- `proscan` — Linux (64-bit)
- `proscan` — macOS (Apple Silicon)

## Step 2: Run

Launch the downloaded executable.

On first run, you'll see the login screen. Sign in with the account you created during purchase or trial registration.

## Step 3: Setup Wizard

After authentication, the setup wizard walks you through initial configuration:

1. **System check** — verifies your machine meets the minimum requirements
2. **Database password** — sets a password for the local database
3. **Admin account** — creates your first admin user for the web interface
4. **Network access** — choose whether the web interface is accessible only locally or over your network
5. **Port configuration** — set the ports for the web UI, database, and cache (defaults work for most setups)
6. **Review** — confirm your settings
7. **Install** — Proscan sets up all required services automatically

The whole process takes a few minutes.

## Step 4: Open the Web Interface

Once setup completes, click "Open Proscan" to launch the web interface in your browser. Log in with the admin credentials you created in step 3.

## Offline / Air-Gapped Installation

For environments without internet access:

1. Activate your license on a machine that has connectivity
2. Download the package file
3. Transfer both to the air-gapped machine
4. Run the executable and use offline activation with your license file

Contact [contact@proscan.one](mailto:contact@proscan.one) for assistance with air-gapped deployments.

## Uninstalling

To remove Proscan, delete the application executable and its data directory. No registry entries or system-level modifications are made during installation.
