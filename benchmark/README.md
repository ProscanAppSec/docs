# Proscan Benchmark Standard

This document defines how Proscan detection quality and performance are measured, validated, and published. It is designed for reproducibility and transparent evaluation by engineering, security, and audit stakeholders.

---

## 1) Benchmark Objectives

The benchmark measures four dimensions:

1. **Detection quality:** what the scanners find, what they miss, and what they incorrectly flag
2. **Performance:** how long scans take at realistic scale
3. **Coverage:** how much of the target surface is actually analyzed
4. **Repeatability:** whether results are stable across identical reruns

## 2) Detection Quality Metrics

### 2.1 Core Metrics

| Metric | Definition |
|--------|-----------|
| True Positives (TP) | Real vulnerabilities correctly identified |
| False Positives (FP) | Non-vulnerabilities incorrectly flagged |
| False Negatives (FN) | Real vulnerabilities missed |
| Precision | TP / (TP + FP) — signal quality |
| Recall | TP / (TP + FN) — detection completeness |
| F1 Score | 2 × (Precision × Recall) / (Precision + Recall) — balanced measure |

### 2.2 Additional Quality Indicators

| Metric | Definition |
|--------|-----------|
| FP Suppression Ratio | Findings eliminated by FP pipeline / total raw matches |
| TP Suppression Rate | True positives incorrectly suppressed / total true positives (target: 0%) |
| Dedup Ratio | Findings removed by deduplication / pre-dedup findings |
| Cross-Scanner Correlation Rate | Findings corroborated by 2+ scanners / total findings |
| Confidence Distribution | Finding count by confidence band (very_high / high / medium / low) |
| Severity Distribution | Finding count by severity (critical / high / medium / low / info) |

## 3) Reference Test Suites

Detection quality should be validated against recognized deliberately-vulnerable applications and industry test corpora.

### 3.1 SAST Validation Targets

| Test Suite | Language | Purpose | Source |
|-----------|---------|---------|--------|
| OWASP WebGoat | Java | Comprehensive web vulnerability training app | github.com/WebGoat/WebGoat |
| OWASP Benchmark | Java | Standardized SAST precision/recall measurement (2,740 test cases) | github.com/OWASP-Benchmark/BenchmarkJava |
| Juliet Test Suite | C/C++, Java | NIST-maintained test cases for CWE coverage (over 80,000 cases) | samate.nist.gov/SARD |
| DVJA (Damn Vulnerable Java App) | Java | Injection, auth, XSS, XXE, deserialization | github.com/appsecco/dvja |
| JavaVulnerableLab | Java | OWASP Top 10 vulnerabilities | github.com/CSPF-Founder/JavaVulnerableLab |
| VulnerableApp | Java | Multi-level vulnerability variants | github.com/SasanLabs/VulnerableApp |
| NodeGoat | JavaScript | OWASP Top 10 for Node.js | github.com/OWASP/NodeGoat |
| Vulnerable Flask App | Python | Python web vulnerabilities | github.com/we45/Vulnerable-Flask-App |
| govwa | Go | Go web application vulnerabilities | github.com/0c34/govwa |
| WrongSecrets | Java/K8s | Secrets management anti-patterns | github.com/OWASP/wrongsecrets |

### 3.2 SCA Validation Targets

| Test Suite | Purpose |
|-----------|---------|
| java-goof | Known-vulnerable Java dependencies (Snyk reference) |
| goof (Node.js) | Known-vulnerable npm dependencies |
| pip-audit test fixtures | Known-vulnerable Python packages |

### 3.3 IaC Validation Targets

| Test Suite | Purpose | Source |
|-----------|---------|--------|
| tfsec test fixtures | Terraform misconfiguration test cases | github.com/aquasecurity/tfsec |
| KICS test fixtures | Multi-format IaC test cases | github.com/Checkmarx/kics |
| Checkov test fixtures | Terraform, CloudFormation, K8s test cases | github.com/bridgecrewio/checkov |

### 3.4 Container Validation Targets

| Test Suite | Purpose |
|-----------|---------|
| CIS Docker Benchmark | Docker daemon and image configuration |
| Dockle test images | Dockerfile best practice violations |
| Trivy test fixtures | Known-vulnerable container images |

### 3.5 DAST and Proscan Deep Validation Targets

| Test Suite | Purpose | Source |
|-----------|---------|--------|
| OWASP WebGoat (running) | Runtime vulnerability validation (SQLi, XSS, XXE, auth, deserialization) | github.com/WebGoat/WebGoat |
| DVWA | Runtime web vulnerability testing across difficulty levels | github.com/digininja/DVWA |
| Juice Shop | Modern JS app with 100+ challenges covering OWASP Top 10 | github.com/juice-shop/juice-shop |
| VulnAPI / vAPI | API-specific vulnerability testing (auth, injection, IDOR) | github.com/roottusk/vapi |
| Hackazon | Realistic e-commerce app for DAST crawling and testing | github.com/rapid7/hackazon |
| bodgeit | Simple vulnerable web app for basic DAST validation | github.com/psiinon/bodgeit |

**Proscan Deep-specific validation:** the 922-check catalog covers 30+ CWE categories. Benchmark should measure per-CWE precision across test targets and validate that all 20 test groups produce findings on appropriate targets. Evidence-based FP controls (SQLi evidence, XSS execution, LFI file content) should be validated to confirm they suppress false positives without suppressing true positives.

### 3.6 AI/LLM Validation

| Test Suite | Purpose |
|-----------|---------|
| OWASP LLM Top 10 test prompts | Coverage validation for LLM01–LLM10 |
| Garak | LLM vulnerability scanner test corpus |
| Internal technique regression suite | 4,600+ technique execution validation |

## 4) Benchmark Profiles

### Quick CI

- **Use case:** pull request and merge gating
- **Scanners:** SAST + SCA + Secrets
- **Scope:** changed files or service-level path filters
- **Success criteria:** no new critical/high findings; scan completes within CI timeout

### Full Baseline

- **Use case:** periodic complete risk assessment
- **Scanners:** all scanners applicable to target type
- **Scope:** full repository, image, or application surface
- **Success criteria:** complete coverage metrics; stable finding count vs. previous baseline

### Compliance Evidence

- **Use case:** audit preparation and control-mapped evidence collection
- **Scanners:** SAST, DAST, SCA, Secrets, IaC, Container, API
- **Scope:** full scan plus compliance report generation
- **Success criteria:** all target frameworks produce non-empty control mappings; export artifacts generated successfully

## 5) Environment Requirements

Every benchmark publication must record:

| Field | Example |
|-------|---------|
| Proscan version | Build ID or release tag |
| OS | Windows Server 2022 / Ubuntu 22.04 / etc. |
| CPU | Model, core count, clock speed |
| RAM | Total capacity |
| Storage | Type (NVMe/SSD/HDD) |
| PostgreSQL version | 15.x |
| Redis version | 7.x |
| Network | Local-only / network assumptions for DAST/API/network scans |
| GOMAXPROCS | Effective value (max(2, numCPU/2)) |
| GOGC | 50 (default) |
| Worker concurrency | 10 (default) |

## 6) Target Pinning

Benchmark targets must use immutable references:

- **Repository:** commit SHA or release tag
- **Container image:** image digest (`sha256:...`)
- **API spec:** versioned spec file (OpenAPI version + hash)
- **Network scope:** documented CIDR/host list snapshot

This ensures reruns test the same target surface.

## 7) Run Protocol

1. **Warm-up run:** execute once to populate caches (rule compilation, tool initialization). Discard results.
2. **Measurement runs:** execute each profile **minimum 3 times** (recommended: 5).
3. **Fixed conditions:** target, profile, and environment must not change between runs.
4. **Record all metrics:** raw finding counts, coverage metrics, runtimes, exported report artifacts.
5. **Report median** as the primary runtime metric. Publish min/median/max for transparency.

## 8) Statistical Rules

- Report **median** for central tendency.
- Report **p95** for long-tail latency when available.
- **Rerun drift flag:** if total finding count changes by more than **5%** between identical runs without target or rule changes, investigate and document the variance source.
- **Severity drift flag:** if any severity bucket changes by more than **10%** without scanner/rule updates, flag for investigation.
- **Determinism expectation:** SAST, SCA, Secrets, IaC, and Container scans should produce identical findings across reruns. DAST and Network scans may have minor variance due to runtime timing.

## 9) Validity Gates

A benchmark run is marked **valid** only if all conditions are met:

- [ ] Profile configuration matches the documented baseline
- [ ] Target reference is unchanged from pinned value
- [ ] All selected scanners complete without critical execution failure
- [ ] Coverage metrics are present and non-empty (files scanned > 0, lines scanned > 0)
- [ ] Export artifacts are generated successfully
- [ ] No environment variable changes between runs (or changes are documented)

## 10) Required Published Fields

| Field | Description |
|-------|-------------|
| Date | Benchmark execution date |
| Proscan Version | Version/build identifier |
| Profile | Quick CI / Full Baseline / Compliance Evidence |
| Target | Target identifier and pinned reference |
| Files Scanned | Total files analyzed |
| Lines Scanned | Total lines processed |
| Languages | Languages detected |
| Rules Applied | Total rules executed |
| Findings | Total and severity breakdown (C/H/M/L/I) |
| Precision | TP / (TP + FP) where ground truth is available |
| Recall | TP / (TP + FN) where ground truth is available |
| Runtime | Min / median / max |
| Correlation Rate | Findings corroborated by 2+ scanners / total |
| Environment | OS, CPU, RAM, storage type |
| Notes | Scanner/rule updates, environment deltas, known issues |

## 11) Current Status

> **Benchmark results have not yet been published.** The methodology, profiles, and validation targets defined in this document are the standard that will be used for all future benchmark publications. Results will be published in [Proscan-hub/scan-results](https://github.com/Proscan-hub/scan-results) as they become available.
>
> When results are published, they will include precision/recall measurements against OWASP Benchmark and other reference test suites listed above, with full environment documentation and raw metric exports.

## 12) Interpretation Guardrails

- Runtime comparisons are valid only within equivalent profile and target classes.
- Higher finding counts do not automatically indicate better detection. Precision and recall are the relevant quality measures.
- Compliance percentages represent technical control evidence coverage, not legal certification.
- Trend interpretation must account for scanner/rule update context between measurement periods.
- SAST precision/recall should be evaluated per CWE category, not just in aggregate, since detection quality varies by vulnerability class.
