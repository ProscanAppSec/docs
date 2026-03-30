# Proscan Documentation

Documentation for [Proscan](https://proscan.one), the on-premises application security platform.

## Contents

### Getting Started
- [System Requirements](getting-started/system-requirements.md)
- [Installation](getting-started/installation.md)
- [Quick Start](getting-started/quick-start.md)
- [Updating](getting-started/updating.md)

### Scanners
- [Overview](scanners/overview.md)
- [SAST — Static Analysis](scanners/sast.md)
- [DAST — Dynamic Testing](scanners/dast.md)
- [ProScan Deep](scanners/proscan-deep.md)
- [SCA — Software Composition Analysis](scanners/sca.md)
- [Secrets Detection](scanners/secrets.md)
- [IaC Security](scanners/iac.md)
- [Container Security](scanners/container.md)
- [API Security](scanners/api.md)
- [AI/LLM Security](scanners/ai-llm.md)
- [Binary Analysis](scanners/binary.md)
- [Network Scanner](scanners/network.md)
- [Code Quality](scanners/code-quality.md)

### Integrations
- [CI/CD Pipelines](integrations/cicd.md)
- [IDE Plugins](integrations/ide.md)
- [Issue Trackers](integrations/issue-trackers.md)
- [Messaging](integrations/messaging.md)
- [SIEM & Monitoring](integrations/siem.md)
- [Webhooks](integrations/webhooks.md)

### Compliance
- [Supported Frameworks](compliance/frameworks.md)
- [Methodology](compliance/methodology.md)
- [Generating Reports](compliance/reports.md)

### Whitepaper and Architecture
- [Whitepaper](whitepaper/README.md)
- [Entire Application Architecture Diagram](architecture/application-architecture.md)
- [Scanner Architecture Diagrams](architecture/scanner-architectures.md)

### Benchmark
- [Benchmark Methodology and Results Template](benchmark/README.md)

### Engineering Proofs
- [Detection Methodology](engineering/detection-methodology.md) — algorithm taxonomy with formal pseudocode
- [Rule Format and Samples](engineering/rule-format.md) — rule schema with representative examples
- [Taint Analysis Model](engineering/taint-analysis.md) — formal source-to-sink propagation specification
- [False Positive Engineering](engineering/fp-engineering.md) — 11-stage suppression pipeline and evidence-based verification
- [Confidence Scoring Model](engineering/confidence-model.md) — 7-factor weighted scoring formula with calibration rationale
- [Detection Coverage Matrix](engineering/coverage-matrix.md) — CWE × scanner coverage table
- [Fuzzing Methodology](engineering/fuzzing-methodology.md) — payload mutation strategies and detection logic
- [Precision and Recall](engineering/precision-recall.md) — measurement methodology and ground-truth test suites

### Sample Reports
- [Overview](sample-reports/README.md)
- [SAST Scan Results](sample-reports/sast-scan-results.json)
- [PCI DSS Compliance Report](sample-reports/pci-dss-compliance.json)
- [Executive Summary](sample-reports/executive-summary.json)
- [SARIF Output](sample-reports/sarif-output.json)
- [CycloneDX SBOM](sample-reports/cyclonedx-sbom.json)

### Report Schemas
- [Scan Results Schema](schemas/scan-results.json)
- [Compliance Report Schema](schemas/compliance-report.json)
- [Executive Summary Schema](schemas/executive-summary.json)

### Administration
- [Users and Roles](administration/users-and-roles.md)
- [Authentication](administration/authentication.md)

## Support

- General: [contact@proscan.one](mailto:contact@proscan.one)
- Security issues: [security@proscan.one](mailto:security@proscan.one)
- Website: [proscan.one](https://proscan.one)
