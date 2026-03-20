# Supported Compliance Frameworks

Proscan maps scan findings to controls in major compliance frameworks. This automates the evidence collection that compliance teams and auditors typically do manually.

## Frameworks

| Framework | Version |
|-----------|---------|
| OWASP Top 10 | 2021 |
| PCI DSS | 4.0 |
| HIPAA | Current |
| SOC 2 | Type II (Trust Services Criteria) |
| NIST 800-53 | Rev 5 |
| NIST CSF | 2.0 |
| ISO 27001 | 2022 |
| GDPR | Current |
| FISMA | Current |
| SOX | Current |
| CISA KEV | Current |
| CERT-C | Current |
| CERT-Java | Current |

## How Mapping Works

Every vulnerability has a CWE (Common Weakness Enumeration) identifier. CWEs map to specific controls in each compliance framework through standardized relationships maintained by MITRE, NIST, and the framework bodies.

When Proscan finds a SQL injection (CWE-89), for example, it automatically links that finding to:

- OWASP A03:2021 (Injection)
- PCI DSS 6.2.4 (Prevention of injection attacks)
- NIST 800-53 SI-10 (Information input validation)

These mappings come from the standards themselves, not from proprietary logic. An auditor can verify any mapping by checking the published framework documentation.

## Control Status

For each framework, Proscan shows:

- Which controls are covered by your scans
- How many findings are associated with each control
- The current status of each control (passing, failing, partial)
- Historical trends over time

This gives compliance teams a live view of their posture instead of a point-in-time snapshot from the last audit.

## Evidence Collection

Proscan collects evidence automatically as scans run. Each control in a framework can be backed by:

- Scan results showing the control was tested
- Finding data showing issues were identified and remediated
- Timestamps proving when testing occurred
- Remediation records showing how quickly issues were resolved

## Questionnaire Auto-Fill

For SOC 2, SIG, CAIQ, and vendor security assessment questionnaires, Proscan includes pre-loaded templates. An answer library maps your scan data to common questionnaire items, reducing the time spent filling out assessments.
