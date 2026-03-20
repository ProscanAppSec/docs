# Issue Trackers

Proscan can create tickets in your issue tracker when vulnerabilities are found. This keeps remediation work in the same system your team already uses for task management.

## Supported Platforms

- Jira
- Linear
- Asana
- Monday
- ClickUp
- Shortcut
- Azure Boards
- ServiceNow
- GitHub Issues
- GitLab Issues

## What Gets Created

When a finding is sent to an issue tracker, the ticket includes:

- Vulnerability title and severity
- Description and CWE identifier
- File location and code reference (for SAST)
- URL and parameter details (for DAST)
- Remediation guidance
- Link back to the finding in Proscan

## Configuration

1. Go to **Settings > Integrations**
2. Select your issue tracker
3. Enter the connection details (API URL, project key, authentication)
4. Configure which findings should create tickets automatically (e.g., only critical and high severity)

You can also create tickets manually from any finding by clicking "Create Ticket" in the finding detail view.

## Two-Way Sync

For supported platforms, ticket status syncs back to Proscan. When a ticket is closed in Jira or marked done in Linear, the corresponding finding in Proscan updates automatically.
