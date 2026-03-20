# SIEM & Monitoring

Proscan can forward findings and scan events to your SIEM or monitoring platform for centralized visibility across your security tooling.

## Supported Platforms

- Splunk
- Datadog
- Elastic (Elasticsearch / Kibana)
- PagerDuty
- Opsgenie
- Sumo Logic

## What Gets Sent

- Scan completion events with finding summaries
- Individual findings (severity, type, location)
- Quality gate pass/fail status
- Audit log events

Data is sent in structured JSON format. For Splunk, Proscan supports HTTP Event Collector (HEC) input directly.

## Setup

1. Go to **Settings > Integrations**
2. Select your SIEM platform
3. Enter the ingestion endpoint and authentication details
4. Configure which event types to forward
5. Test the connection

## Alerting

Once findings flow into your SIEM, you can build alerts using your existing monitoring rules. For example, trigger a PagerDuty alert when a critical finding is detected in a production scan, or create a Datadog dashboard tracking vulnerability trends across all scanned projects.
