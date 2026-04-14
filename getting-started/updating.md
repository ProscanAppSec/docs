# Updating Proscan

## How Updates Work

Proscan updates replace only the backend images. All your data is preserved:
- Scan results and findings
- Project configurations
- User accounts and permissions
- Compliance reports
- Triage decisions and notes

## Check for Updates

From the launcher dashboard, click **"Check Updates"**. The launcher checks for new versions automatically via periodic heartbeats.

## Apply an Update

1. Click **"Update"** when a new version is available
2. The launcher stops running services
3. The new backend image is downloaded and verified
4. Services restart with the updated version
5. Database migrations run automatically if needed

The update process takes 1-3 minutes depending on your connection speed.

## Rollback

If an update causes issues, contact [support@proscan.one](mailto:support@proscan.one) for rollback assistance. Your data is preserved across updates, so rolling back to a previous version is straightforward.

## Backup Before Updating

It's good practice to create a backup before applying updates:

1. Go to the launcher dashboard
2. Click **"Create Backup"**
3. Set a password for the backup file
4. Save the `.gpbak` file

Backups can be restored from the same dashboard.
