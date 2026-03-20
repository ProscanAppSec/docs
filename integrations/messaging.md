# Messaging Integrations

Proscan sends notifications to your team's chat platform when scans complete, critical findings are discovered, or other configured events occur.

## Supported Platforms

- Slack
- Microsoft Teams
- Discord
- Mattermost
- Google Chat
- Telegram

## Notification Events

You can configure which events trigger notifications:

- Scan started
- Scan completed (with summary)
- Critical or high severity finding discovered
- Quality gate passed or failed
- Scheduled scan results available

## Setup

1. Go to **Settings > Integrations**
2. Select your messaging platform
3. Enter the webhook URL (each platform provides instructions for creating incoming webhooks)
4. Choose which events to send notifications for
5. Optionally choose a specific channel or thread

## Message Format

Notifications include a summary of the event — for scan completions, that means the number of findings by severity, pass/fail status of quality gates, and a link to view full results in Proscan.
