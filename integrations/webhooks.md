# Webhooks

For integrations not covered by built-in connectors, Proscan supports generic webhooks. Any event can trigger an HTTP POST to a URL you specify.

## How It Works

1. Go to **Settings > Integrations > Webhooks**
2. Add a webhook URL
3. Select the events that should trigger it
4. Save

When a matching event occurs, Proscan sends a POST request to your URL with a JSON payload containing the event details.

## Available Events

- Scan started
- Scan completed
- Finding created
- Quality gate evaluation (pass/fail)
- Scan schedule triggered

## Payload

The payload includes:

- Event type
- Timestamp
- Relevant data (scan details, finding details, quality gate results)

The exact schema depends on the event type. All payloads include an HMAC signature header so you can verify the request came from your Proscan instance.

## Retry Behavior

If the webhook endpoint returns a non-2xx response, Proscan retries the delivery with exponential backoff. Failed deliveries are logged and visible in the webhook settings page.

## Use Cases

- Trigger custom automation when a scan finishes
- Feed findings into an internal dashboard or reporting system
- Notify systems not supported by built-in integrations
- Kick off remediation workflows in custom tooling
