# Compliance Methodology

This page describes how Proscan maps scan findings to compliance framework controls and generates audit-ready reports.

## How Findings Map to Controls

Every vulnerability Proscan identifies carries a CWE (Common Weakness Enumeration) identifier. CWE is an industry-standard classification maintained by MITRE that catalogs software weakness types.

Each compliance framework defines controls that relate to specific types of software weaknesses. These relationships are well-documented in the framework publications themselves, in MITRE's CWE cross-references, and in NIST's National Vulnerability Database.

Proscan uses these published relationships to link each finding to the relevant controls. The mappings are available at [Proscan-hub/compliance-mappings](https://github.com/Proscan-hub/compliance-mappings) for inspection.

### Example

A SQL Injection finding (CWE-89) maps to:

| Framework | Control | Rationale |
|-----------|---------|-----------|
| OWASP Top 10 | A03:2021 — Injection | CWE-89 is explicitly listed under this category |
| PCI DSS 4.0 | 6.2.4 | Requires prevention of injection attacks in custom software |
| NIST 800-53 | SI-10 | Requires input validation to prevent injection |
| HIPAA | §164.312(c)(1) | Requires technical safeguards for data integrity |
| ISO 27001 | A.8.28 | Requires secure coding practices including injection prevention |

None of these mappings are proprietary. They follow the same standards that any security tool or auditor would use.

## What a Compliance Report Contains

Each report covers a single framework and includes:

- **Control listing** — every control relevant to application security, with its ID and title
- **Control status** — passing, failing, or not applicable, based on scan results
- **Evidence** — the findings linked to each control, with severity, location, and remediation status
- **Remediation timeline** — when findings were discovered and how quickly they were resolved
- **Summary statistics** — total controls, pass/fail counts, compliance score, and mean time to remediate

The report does not make a compliance determination on its own. It provides the technical evidence that an auditor or compliance team uses as part of their broader assessment.

## What Proscan Covers

Proscan covers **technical controls** — the controls that relate to software vulnerabilities, coding practices, encryption, access control, logging, and infrastructure configuration.

Most compliance frameworks also include administrative, physical, and operational controls that fall outside the scope of automated scanning. These include security policies, employee training, physical access restrictions, incident response plans, and vendor management.

A complete compliance assessment requires both automated evidence (what Proscan provides) and manual evidence for the controls that software scanning cannot address.

## Scanner-to-Control Coverage

Different scanners within Proscan provide evidence for different types of controls:

| Scanner | Types of Controls Covered |
|---------|--------------------------|
| SAST | Secure coding, input validation, injection prevention, cryptographic usage |
| DAST | Application-level access control, authentication, transport security, injection |
| SCA | Vulnerability management, software inventory, patch management |
| Secrets | Credential management, authenticator handling |
| IaC | Configuration management, infrastructure hardening, CIS benchmarks |
| Container | Image security, runtime configuration, patching |
| API | API access control, authentication, injection, rate limiting |
| Network | Vulnerability scanning, service hardening, CVE identification |

Running multiple scanner types against the same target provides broader control coverage than any single scan type alone.

## Report Formats

Compliance reports can be exported in:

- **PDF** — for sharing with auditors and attaching to compliance packages
- **HTML** — for interactive review within the organization
- **JSON** — for programmatic consumption and integration with GRC tools
- **CSV** — for import into spreadsheets
- **XML** — for integration with compliance automation platforms
- **SARIF** — for GitHub/GitLab security dashboard integration

## Audit Packages

For formal audit engagements, Proscan can generate an audit package that combines:

- Framework control mapping with status
- Evidence from scan results for each control
- Finding history with discovery and remediation dates
- Summary metrics (mean time to remediate, finding trends, false positive rate)
- Scan coverage and schedule information

This package is designed to give an auditor a complete view of the organization's technical security posture for the relevant framework.

## Verification

All mappings used by Proscan are derived from publicly available standards. Auditors and compliance teams can verify any mapping by:

1. Looking up the CWE identifier at [cwe.mitre.org](https://cwe.mitre.org/)
2. Checking the relevant framework's documentation for that control
3. Confirming that the CWE category falls within the control's scope

The mappings are also published at [Proscan-hub/compliance-mappings](https://github.com/Proscan-hub/compliance-mappings) for open review.
