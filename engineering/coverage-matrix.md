# Detection Coverage Matrix

This document maps CWE vulnerability categories to Proscan scanner modules. It shows which scanners detect which vulnerability classes and how detection methods differ across scanners.

---

## 1) CWE × Scanner Coverage

| CWE | Vulnerability | SAST | DAST | Deep | SCA | Secrets | IaC | Container | API | AI/LLM |
|-----|--------------|------|------|------|-----|---------|-----|-----------|-----|--------|
| CWE-22 | Path Traversal | ✓ | ✓ | ✓ | | | | | ✓ | |
| CWE-74 | Injection (General) | ✓ | | ✓ | | | | | | |
| CWE-78 | OS Command Injection | ✓ | ✓ | ✓ | | | | | ✓ | |
| CWE-79 | Cross-Site Scripting | ✓ | ✓ | ✓ | | | | | ✓ | |
| CWE-89 | SQL Injection | ✓ | ✓ | ✓ | | | | | ✓ | |
| CWE-90 | LDAP Injection | ✓ | ✓ | ✓ | | | | | | |
| CWE-91 | JSON Injection | | ✓ | ✓ | | | | | | |
| CWE-93 | Email Header Injection | | ✓ | | | | | | | |
| CWE-94 | Code/Template Injection | ✓ | ✓ | ✓ | | | | | | |
| CWE-97 | SSI Injection | | ✓ | | | | | | | |
| CWE-98 | Remote File Inclusion | ✓ | ✓ | ✓ | | | | | | |
| CWE-113 | HTTP Header Injection | ✓ | | ✓ | | | | | | |
| CWE-117 | Log Injection | ✓ | | | | | | | | |
| CWE-176 | Unicode Normalization | | ✓ | | | | | | | |
| CWE-200 | Information Exposure | ✓ | ✓ | ✓ | | | ✓ | ✓ | ✓ | |
| CWE-235 | Parameter Pollution | | ✓ | ✓ | | | | | | |
| CWE-284 | Authorization Bypass | | | ✓ | | | | | ✓ | |
| CWE-287 | Authentication Bypass | ✓ | ✓ | ✓ | | | | | ✓ | |
| CWE-295 | Certificate Validation | | | ✓ | | | ✓ | ✓ | | |
| CWE-310 | Cryptographic Issues | ✓ | | ✓ | | | ✓ | | | |
| CWE-327 | Weak Cryptography | ✓ | | | | | | | | |
| CWE-347 | JWT Algorithm Confusion | | ✓ | ✓ | | | | | ✓ | |
| CWE-350 | IP Spoofing / Host Forgery | | | ✓ | | | | | | |
| CWE-352 | Cross-Site Request Forgery | ✓ | ✓ | ✓ | | | | | | |
| CWE-362 | Race Condition | | | ✓ | | | | | | |
| CWE-434 | Unrestricted File Upload | ✓ | ✓ | ✓ | | | | | | |
| CWE-444 | HTTP Request Smuggling | | | ✓ | | | | | | |
| CWE-489 | Debug Mode Active | | | ✓ | | | ✓ | ✓ | | |
| CWE-502 | Insecure Deserialization | ✓ | ✓ | ✓ | | | | | | |
| CWE-540 | Source Code Exposure | | | ✓ | | | | | | |
| CWE-564 | HQL Injection | | ✓ | | | | | | | |
| CWE-601 | Open Redirect | ✓ | ✓ | ✓ | | | | | | |
| CWE-611 | XML External Entity (XXE) | ✓ | ✓ | ✓ | | | | | | |
| CWE-639 | IDOR | | | ✓ | | | | | ✓ | |
| CWE-643 | XPath Injection | | | ✓ | | | | | | |
| CWE-644 | Improper HTTP Headers | | | ✓ | | | | | | |
| CWE-693 | Protection Mechanism Failure | | | ✓ | | | | | | |
| CWE-776 | XML Bomb | | ✓ | | | | | | | |
| CWE-843 | Type Confusion | | ✓ | | | | | | | |
| CWE-918 | SSRF | ✓ | ✓ | ✓ | | | | | ✓ | |
| CWE-942 | Permissive CORS | | ✓ | ✓ | | | | | ✓ | |
| CWE-943 | NoSQL Injection | ✓ | | ✓ | | | | | | |
| CWE-1021 | Clickjacking | | | ✓ | | | ✓ | | | |
| CWE-1333 | ReDoS | | ✓ | ✓ | | | | | | |

**Note:** SAST covers **500+ CWEs** total through its 200,000+ rule database. This matrix shows the highest-impact categories. SCA coverage is CVE-based (mapping to NVD/OSV vulnerability identifiers rather than CWE patterns). Secrets scanner detects credential exposure (CWE-798, CWE-312, CWE-321) across 17 secret types. Container scanner detects configuration issues aligned to CIS Docker Benchmark.

## 2) Cross-Scanner Detection Depth

Some vulnerabilities are detected by multiple scanners using different methods:

| Vulnerability | SAST Method | DAST/Deep Method | Correlation Benefit |
|--------------|-------------|------------------|-------------------|
| SQL Injection | Pattern matching + taint analysis on source code | Active payload injection + response differential | SAST finds the code path; DAST confirms exploitability |
| XSS | Pattern matching + output encoding checks | Payload reflection in executable context | SAST finds missing encoding; DAST proves script execution |
| SSRF | Taint analysis (URL construction from user input) | OOB callback / response analysis | SAST finds the sink; DAST confirms reachability |
| Deserialization | Pattern matching (readObject, pickle.loads) | Payload injection with gadget chain | SAST identifies the deserializer; DAST tests exploitation |
| Path Traversal | Taint analysis (file path from user input) | File content signature in response | SAST finds the code; DAST confirms file read |
| Hardcoded Secrets | SAST rule patterns + Secrets entropy analysis | — | Two independent detection methods increase confidence |

## 3) Language × Vulnerability Coverage (SAST)

SAST rules cover **60 language categories** across **221 file extensions**. The highest-density rule coverage is in:

| Language | Primary Vulnerability Focus |
|----------|-----------------------------|
| Java | SQLi, XSS, XXE, deserialization, SSRF, command injection, path traversal, LDAP injection, open redirect, weak cryptography |
| JavaScript/TypeScript | XSS, prototype pollution, command injection, path traversal, SSRF, open redirect, NoSQL injection |
| Python | SQLi, command injection, SSRF, path traversal, deserialization, template injection, code injection |
| Go | SQLi, command injection, SSRF, path traversal, XSS, log injection |
| C# | SQLi, XSS, command injection, deserialization, path traversal, XXE |
| PHP | SQLi, XSS, command injection, file inclusion, path traversal, deserialization |
| Ruby | SQLi, XSS, command injection, open redirect, deserialization |
| Kotlin | SQLi, XSS, SSRF (inherits Java ecosystem rules + Kotlin-specific patterns) |
| Rust | Command injection, path traversal, unsafe operations |
| Swift | SQLi, path traversal, command injection |

## 4) Proscan Deep Check Distribution

The Deep scanner's 922 checks are distributed across these CWE categories:

| CWE | Check Count (approximate) | Category |
|-----|--------------------------|----------|
| CWE-200 | High | Information Exposure |
| CWE-287 | High | Authentication |
| CWE-918 | High | SSRF |
| CWE-502 | High | Deserialization |
| CWE-79 | High | XSS |
| CWE-89 | Medium | SQL Injection |
| CWE-22 | Medium | Path Traversal |
| CWE-611 | Medium | XXE |
| CWE-98 | Medium | Remote File Inclusion |
| CWE-601 | Low | Open Redirect |
| CWE-78 | Low | Command Injection |
| CWE-350 | Low | IP Spoofing |
| Others | Low | Various (30+ CWE IDs total) |

## 5) Compliance Framework Mapping

Detection coverage maps to compliance frameworks through CWE-to-control relationships:

| Framework | Relevant CWE Categories | Scanner Coverage |
|-----------|------------------------|-----------------|
| OWASP Top 10 2021 | A01 (Access Control), A02 (Crypto), A03 (Injection), A04 (Design), A05 (Config), A06 (Components), A07 (Auth), A08 (Integrity), A09 (Logging), A10 (SSRF) | SAST + DAST + Deep + SCA + Secrets + IaC + Container |
| PCI DSS 4.0 | 6.2.4 (Injection), 6.2.3 (Crypto), 6.2.2 (XSS), 6.3.1 (Vulns), 6.3.2 (Patching) | SAST + DAST + SCA |
| NIST 800-53 | SI-10 (Input Validation), SC-13 (Crypto), CM-6 (Config), RA-5 (Scanning) | All scanners |
| HIPAA | §164.312 (Technical Safeguards) | SAST + SCA + Secrets + IaC |
| SOC 2 | CC6.1 (Logical Access), CC7.1 (Monitoring), CC8.1 (Change Management) | SAST + SCA + IaC |
