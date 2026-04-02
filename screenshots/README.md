# Proscan Screenshots and Videos

Real screenshots and video clips from Proscan v9.0 Enterprise. All content shows actual scan results on real targets.

---

## Video: Product Overview (21 seconds)

<video src="videos/proscan-overview-21s.mp4" width="100%" controls></video>

## Video: Product Highlights (83 seconds)

<video src="videos/proscan-highlights-83s.mp4" width="100%" controls></video>

---

## Security Command Center

### Video: Dashboard Preview (7 seconds)

<video src="videos/dashboard-preview-7s.mp4" width="100%" controls></video>

The unified dashboard shows your security posture across all active scanners — total findings, severity breakdown, 7-day trend, and real-time scanner status.

![Security Command Center](dashboard-security-command-center.png)

---

## SAST — Static Analysis

### Video: SAST Dashboard (30 seconds)

<video src="videos/sast-dashboard-30s.mp4" width="100%" controls></video>

### Configure and Start a Scan

Browse a local folder, connect to a remote Git repository, or upload an archive. Choose from Standard, Quick Scan, Deep Analysis, or OWASP Top 10 profiles.

![SAST Configure](sast-dashboard-configure.png)

### Scan History

Every scan records files scanned, findings count, and duration. Full history is retained for trend analysis.

![SAST Scan History](sast-scan-history.png)

### Findings List

Findings are listed by severity with CWE classification, file location, and status. Filter by severity, type, or status.

![SAST Findings](sast-findings-list.png)

### Finding Detail with Auto-Fix

Each finding includes the vulnerable code snippet, confidence score, rule reference, remediation recommendation, and a one-click Auto-Fix Engine that generates a fix suggestion.

![SAST Finding Detail](sast-finding-detail-autofix.png)

---

## Binary & Bytecode Analysis

### Supported Formats

Analyzes JVM Bytecode (.jar/.class), .NET IL, Android DEX, ELF (Linux), PE (Windows), and Mach-O (macOS). Taint analysis with SSA and CHA call graph construction.

![Binary Analysis Dashboard](binary-analysis-dashboard.png)

### Analysis Summary

For each binary, the engine reports: classes parsed, methods analyzed, SSA methods, call graph edges, source patterns, sink patterns, and sanitizers identified.

![Binary Analysis Summary](binary-analysis-findings-summary.png)

### Findings with CWE Mapping

Every finding includes severity, vulnerability category, detection method, and CWE identifier.

![Binary Analysis Findings](binary-analysis-findings-list.png)

### Finding Detail with Remediation Steps

Each finding includes taint source → sink path, file and offset location, remediation recommendation, and numbered remediation steps with compiler flags and safe API alternatives.

![Binary Finding Detail](binary-finding-detail-remediation.png)

### Cryptographic Algorithm Analysis

Identifies all cryptographic algorithms used in the binary — algorithm type, key size, strength (weak/strong/acceptable), occurrences, and binary offset.

![Binary Crypto Analysis](binary-crypto-analysis.png)

---

## DAST — Dynamic Analysis

### Video: Live Scan Progress (19 seconds)

<video src="videos/dast-live-progress-19s.mp4" width="100%" controls></video>

### Video: Deep Scan Findings (34 seconds)

<video src="videos/dast-deep-findings-34s.mp4" width="100%" controls></video>

### Configure Target

Point at any URL with Active Scan, Passive Scan, or both. Advanced options for authentication, custom headers, scope, and rate limiting.

![DAST Configure](dast-configure.png)

### Deep Scan Metrics (Proscan Deep)

Real-time scan metrics: progress percentage, URLs discovered, parameters tested, payloads fired, tests executed, and vulnerabilities found. Live speed adjustment with dynamic RPS control.

![DAST Deep Scan Metrics](dast-deep-scan-metrics.png)

### Deep Scan Findings

Findings listed with severity, verification status, URL, and parameter. Verified badge indicates findings confirmed by the engine (not just detected).

![DAST Deep Findings](dast-deep-findings.png)

---

## AI/LLM Security Scanner

Tests AI chatbot endpoints against 1,537 prompt injection techniques (OWASP LLM Top 10 coverage). Supports OpenAI, Anthropic, and any API-compatible endpoint.

![AI LLM Scanner](ai-llm-scanner.png)
