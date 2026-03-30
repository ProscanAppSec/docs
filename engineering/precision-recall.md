# Precision and Recall Measurement

This document defines how Proscan detection quality is measured and validated against industry-standard ground-truth test suites.

---

## 1) Metrics Definitions

| Metric | Formula | What It Measures |
|--------|---------|-----------------|
| **True Positive (TP)** | Finding matches a known real vulnerability | Correct detection |
| **False Positive (FP)** | Finding reported where no vulnerability exists | Incorrect detection (noise) |
| **False Negative (FN)** | Known vulnerability not detected | Missed detection (gap) |
| **True Negative (TN)** | No finding where no vulnerability exists | Correct silence |
| **Precision** | TP / (TP + FP) | Signal quality — what fraction of reported findings are real |
| **Recall** | TP / (TP + FN) | Coverage — what fraction of real vulnerabilities are found |
| **F1 Score** | 2 × (Precision × Recall) / (Precision + Recall) | Balanced measure of both |
| **False Positive Rate** | FP / (FP + TN) | Noise rate |
| **TP Suppression Rate** | Suppressed TP / Total TP | Pipeline defect rate (target: 0%) |

## 2) Ground-Truth Test Suites

### 2.1 OWASP Benchmark (SAST Primary Benchmark)

- **Source:** [github.com/OWASP-Benchmark/BenchmarkJava](https://github.com/OWASP-Benchmark/BenchmarkJava)
- **Test cases:** 2,740 Java test cases
- **Ground truth:** `expectedresults-1.2.csv` — each test case labeled as true positive or false positive
- **CWE categories covered:** CWE-22 (Path Traversal), CWE-78 (Command Injection), CWE-79 (XSS), CWE-89 (SQL Injection), CWE-90 (LDAP Injection), CWE-327 (Weak Crypto), CWE-328 (Weak Hash), CWE-330 (Weak Random), CWE-501 (Trust Boundary), CWE-614 (Cookie Security), CWE-643 (XPath Injection)
- **Industry standard:** Checkmarx, Fortify, SonarQube, Semgrep, and Snyk all publish scores against this benchmark
- **Scoring methodology:** standardized TPR (True Positive Rate) and FPR (False Positive Rate) per CWE category

### 2.2 Juliet Test Suite (NIST SARD)

- **Source:** [samate.nist.gov/SARD](https://samate.nist.gov/SARD/test-suites/111)
- **Test cases:** 80,000+ test cases across C/C++ and Java
- **Ground truth:** each test case has a documented CWE and a known good/bad variant
- **Purpose:** broader CWE coverage validation than OWASP Benchmark

### 2.3 Deliberately Vulnerable Applications

| Application | Language | Vuln Types | Source |
|-------------|---------|------------|--------|
| OWASP WebGoat | Java | SQLi, XSS, XXE, auth, deserialization, SSRF, path traversal | github.com/WebGoat/WebGoat |
| DVWA | PHP | SQLi, XSS, CSRF, file inclusion, command injection | github.com/digininja/DVWA |
| Juice Shop | JavaScript | OWASP Top 10, 100+ challenges | github.com/juice-shop/juice-shop |
| NodeGoat | JavaScript | OWASP Top 10 for Node.js | github.com/OWASP/NodeGoat |
| VulnerableApp | Java | Multi-level vulnerability variants | github.com/SasanLabs/VulnerableApp |
| govwa | Go | SQLi, XSS, IDOR for Go | github.com/0c34/govwa |
| WrongSecrets | Java/K8s | Secrets management anti-patterns | github.com/OWASP/wrongsecrets |

## 3) Scoring Methodology

### 3.1 OWASP Benchmark Scoring

```
For each CWE category C:
  1. Run Proscan SAST scan on Benchmark source code
  2. Map each finding to its Benchmark test case ID
  3. Compare against expectedresults-1.2.csv:
     - Finding on a "true" test case → TP
     - Finding on a "false" test case → FP
     - No finding on a "true" test case → FN
     - No finding on a "false" test case → TN
  4. Calculate per-CWE metrics:
     TPR = TP / (TP + FN)
     FPR = FP / (FP + TN)
     Precision = TP / (TP + FP)
     F1 = 2 × TPR × Precision / (TPR + Precision)
```

### 3.2 Deliberately Vulnerable Application Scoring

```
For each application A:
  1. Compile ground truth:
     - Documented vulnerabilities from application README/docs
     - Manually verified vulnerability inventory
  2. Run Proscan scan (SAST, DAST, or both depending on app type)
  3. Map findings to ground truth inventory
  4. Score: TP, FP, FN, precision, recall, F1
```

### 3.3 Per-CWE Reporting

Results are always broken down **per CWE category**, not just in aggregate. This is critical because:
- A scanner might have 95% overall precision but 60% precision for XSS specifically
- Per-CWE metrics reveal which vulnerability classes need rule improvement
- Auditors and evaluators expect CWE-level granularity

## 4) Result Format

### Per-CWE Scorecard

| CWE | Category | TP | FP | FN | TN | Precision | Recall (TPR) | FPR | F1 |
|-----|----------|----|----|----|----|-----------|-------------|-----|-----|
| CWE-89 | SQL Injection | — | — | — | — | —% | —% | —% | — |
| CWE-79 | XSS | — | — | — | — | —% | —% | —% | — |
| CWE-78 | Command Injection | — | — | — | — | —% | —% | —% | — |
| CWE-22 | Path Traversal | — | — | — | — | —% | —% | —% | — |
| CWE-90 | LDAP Injection | — | — | — | — | —% | —% | —% | — |
| CWE-327 | Weak Crypto | — | — | — | — | —% | —% | —% | — |
| CWE-328 | Weak Hash | — | — | — | — | —% | —% | —% | — |
| CWE-330 | Weak Random | — | — | — | — | —% | —% | —% | — |
| CWE-501 | Trust Boundary | — | — | — | — | —% | —% | —% | — |
| CWE-614 | Cookie Security | — | — | — | — | —% | —% | —% | — |
| CWE-643 | XPath Injection | — | — | — | — | —% | —% | —% | — |
| **ALL** | **Aggregate** | **—** | **—** | **—** | **—** | **—%** | **—%** | **—%** | **—** |

### Aggregate Summary

| Metric | Value |
|--------|-------|
| Total Test Cases | — |
| True Positives | — |
| False Positives | — |
| False Negatives | — |
| Overall Precision | —% |
| Overall Recall | —% |
| Overall F1 | — |
| TP Suppression Rate | —% (target: 0%) |

## 5) Current Status

> **Benchmark results are in preparation.** The scoring methodology and ground-truth test suites are defined. Results will be published as they are validated. Each publication will include:
>
> - Proscan version and build identifier
> - Test suite version and commit SHA
> - Complete per-CWE scorecard
> - Aggregate metrics
> - Environment specification
> - Raw finding export for independent verification
>
> Results will be published in the [scan-results](https://github.com/Proscan-hub/scan-results) repository.

## 6) Comparison Context

The OWASP Benchmark project maintains a [public scorecard](https://owasp.org/www-project-benchmark/) showing results from major commercial and open-source SAST tools. Proscan results, when published, can be directly compared against these published scores using the same TPR/FPR methodology.

## 7) Continuous Validation

Precision/recall is not a one-time measurement. It is validated:

- **Per release:** each Proscan version is scored against the same benchmark suite
- **Per rule change:** new rules or suppression logic changes trigger regression testing
- **Per engine change:** taint analysis, confidence scoring, or FP pipeline changes require full re-validation
- **Trend tracking:** precision and recall are tracked over time to ensure they improve or remain stable with each release
