# Generating Compliance Reports

Proscan generates audit-ready reports that map your scan findings to compliance framework controls. These reports are designed to be handed directly to auditors or attached to compliance documentation.

## Report Types

- **Framework-specific** — a report focused on a single framework (e.g., PCI DSS 4.0), showing every relevant control and its status
- **Unified** — a consolidated report across all active scanners and all mapped frameworks
- **Executive summary** — a high-level overview of security posture, trends, and risk areas for non-technical stakeholders

## Output Formats

| Format | Use Case |
|--------|----------|
| PDF | Share with auditors, attach to compliance packages |
| HTML | Interactive viewing, internal distribution |
| JSON | Feed into other systems, automated processing |
| CSV | Import into spreadsheets for custom analysis |
| XML | Integration with GRC platforms |
| SARIF | GitHub/GitLab security dashboard integration |

## Generating a Report

1. Go to **Compliance**
2. Select the framework you need a report for
3. Review the control status and evidence
4. Click **Generate Report**
5. Choose the output format
6. Download

## Audit Packages

For formal audits, you can generate an audit package that bundles:

- Framework control mapping
- Evidence from scan results
- Finding history with remediation timelines
- Summary statistics (mean time to remediate, finding trends, false positive rates)

This gives the auditor everything they need in a single deliverable rather than pulling data from multiple places.

## Scheduling

Reports can be generated on a schedule — weekly, monthly, or quarterly — and sent to a distribution list automatically. This is useful for recurring compliance reviews or management reporting.
