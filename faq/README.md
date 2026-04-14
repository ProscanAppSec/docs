# Frequently Asked Questions

---

## General

**What is Proscan?**

Proscan is an on-premises application security platform that consolidates 12 scanner modules — SAST, DAST, SCA, Secrets Detection, IaC, Container, API Security, AI/LLM Security, Binary Analysis, Network Scanning, Code Quality, and Proscan Deep — into a single tool with unified reporting.

**Is it open source?**

No. Proscan is proprietary software that runs on your infrastructure. The technical documentation, engineering methodology, benchmark results, and sample scan reports are published publicly on GitHub.

**Does it require an internet connection?**

No. Once installed, Proscan runs entirely offline. There is no telemetry, no cloud dependency, and no data leaving your environment. The only external connection is an optional license validation at startup.

**Where does my source code go?**

It stays on your machine. Proscan scans local directories, uploaded archives, or connected Git repositories — all processed locally. Nothing is transmitted to external servers.

---

## Accuracy and Detection

**How do I know the results are accurate?**

Proscan's SAST engine was validated against [OWASP Benchmark v1.2](https://owasp.org/www-project-benchmark/) — an industry-standard test suite with 2,740 Java test cases and a published ground truth file. The results: 1,415 TP, 0 FP, 0 FN, 1,325 TN. Full scorecard is published at [github.com/Proscan-hub/docs](https://github.com/Proscan-hub/docs/blob/main/benchmark/results/OWASP_BENCHMARK_SCORECARD.md).

**Can I verify the benchmark score independently?**

Yes. The OWASP Benchmark project is publicly available at [github.com/OWASP-Benchmark/BenchmarkJava](https://github.com/OWASP-Benchmark/BenchmarkJava). Download it, run Proscan against it, and compare your results against `expectedresults-1.2.csv`. The methodology is published at [github.com/Proscan-hub/docs/engineering/precision-recall.md](https://github.com/Proscan-hub/docs/blob/main/engineering/precision-recall.md).

**What is the false positive rate on real-world code?**

OWASP Benchmark measures false positive rate on synthetic Java test cases. Real-world FP rates vary by codebase complexity, language, and framework usage. Our confidence scoring and multi-layer suppression pipeline are designed to minimize FP across real codebases. We offer demos for organizations who want to evaluate on their own code.

**Does the OWASP Benchmark score apply to all languages?**

The OWASP Benchmark is Java-only. We are working on validation for Python, JavaScript, and Go using the Juliet Test Suite. The same engine architecture and analysis techniques apply across all supported languages.

**What's the difference between SAST, DAST, and Proscan Deep?**

- **SAST** analyzes source code without executing it. Best for finding vulnerabilities in your own codebase.
- **DAST** tests a running application from the outside. Best for finding vulnerabilities in deployed web applications.
- **Proscan Deep** is an advanced dynamic scanner that combines reconnaissance, multi-step attack simulation, and exploitability validation. Think of it as an automated penetration tester.

---

## Technical

**What languages does SAST support?**

60+ languages including Java, JavaScript, TypeScript, Python, Go, C#, PHP, Ruby, Kotlin, Swift, Rust, Scala, C, C++, Terraform, Kubernetes YAML, and more. See the full [Language Support Matrix](../language-support/README.md).

**What CWEs does Proscan detect?**

500+ CWE categories across all scanners. See the full [Supported CWE Catalog](../supported-cwes/README.md).

**Does it integrate with CI/CD pipelines?**

Yes. GitHub Actions, GitLab CI, Jenkins, Azure Pipelines, CircleCI, and more. See the [CI/CD Integration Guide](../integrations/cicd.md).

**What report formats are supported?**

HTML, PDF, JSON, CSV, SARIF (for GitHub/GitLab Security dashboards), and XML. Four visual themes: dark, light, blue, terminal.

**How does it handle large codebases?**

Proscan uses parallel scanning with adaptive concurrency. It processes up to 8 files concurrently for SAST with a cap of 500 findings per file and 10,000 findings per scan to prevent noise from overwhelming large codebases.

---

## Evaluation and Purchase

**How do I request a demo?**

Contact us at [contact@proscan.one](mailto:contact@proscan.one) or visit [proscan.one](https://proscan.one). We offer demos for organizations evaluating the tool on their own codebase.

**How do I evaluate it fairly against other tools?**

See our [Evaluation Guide](../evaluation-guide/README.md) for methodology, test targets, and metrics.

**Is there a trial version?**

We offer a 15-day free trial — see the [download page](https://proscan.one/download) for details.
