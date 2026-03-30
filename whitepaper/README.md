# Proscan Technical Whitepaper

Version: 1.0  
Classification: Public  
Audience: Security engineering, AppSec, platform engineering, compliance, and audit stakeholders

---

## 1) Executive Summary

Proscan is an on-premises application security platform that consolidates twelve scanner modules into a single execution, normalization, and reporting pipeline. It is designed for organizations that need broad vulnerability coverage across their entire application stack without sending source code or scan artifacts to external SaaS providers.

The platform ships with:

- **200,000+ detection rules** across SAST alone (420 rule sets, 500+ CWE categories)
- **221 file extensions** supported for static analysis
- **173 dependency parsers** covering 16 package ecosystems
- **2,200+ IaC security rules** across Terraform, Kubernetes, Dockerfile, CloudFormation, Helm, and Ansible
- **200+ container security checks** aligned to CIS Docker Benchmark
- **150+ DAST security checks** across 7 enterprise categories plus CVE-specific detection
- **4,600+ AI/LLM security techniques** with full OWASP LLM Top 10 (LLM01–LLM10) coverage
- **34 secrets detection patterns** plus Shannon entropy analysis
- **12 cross-scanner correlation rules** for multi-evidence confidence elevation
- **6 report formats** (HTML, PDF, JSON, CSV, SARIF, XML) with 4 visual themes and 6 report styles

## 2) Problem Definition

Most organizations run multiple disconnected security tools. This creates three measurable issues:

1. **Schema fragmentation:** each tool produces findings in its own format with its own severity model, making centralized triage expensive.
2. **Redundant validation:** security teams re-verify overlapping findings across tools without shared context, wasting analyst time.
3. **Audit assembly cost:** compliance evidence packages require manual stitching from disparate tool outputs, introducing delay and error risk.

Proscan addresses this by standardizing the execution model, the vulnerability schema, the prioritization logic, and the reporting output across all scanner types.

## 3) Scanner Modules

| Module | Detection Focus | Key Scale Metric |
|--------|----------------|-----------------|
| SAST | Source code vulnerabilities across 60+ languages | 200,000+ rules, 500+ CWEs, 221 file extensions |
| DAST | Runtime web application vulnerabilities | 150+ checks across 7 enterprise categories |
| Proscan Deep (Elite) | Full-lifecycle dynamic security testing with crawling, payload injection, and exploitation validation | 922 security checks, 30+ CWE categories, 20 test groups, 605 Go source files |
| SCA | Open-source dependency vulnerabilities and license risk | 173 parsers, 16 ecosystems, OSV + NVD + EPSS + KEV |
| Secrets | Hardcoded credentials, API keys, tokens | 34 pattern rules + entropy analysis (threshold 4.5) |
| IaC | Infrastructure-as-code misconfigurations | 2,200+ rules (Terraform, K8s, Docker, CFN, Helm, Ansible) |
| Container | Image vulnerabilities and Dockerfile analysis | 200+ checks, 80 Dockerfile rules, CIS Docker Benchmark |
| API | REST and GraphQL endpoint security | Schema-driven auth, injection, and logic abuse testing |
| AI/LLM | LLM and AI model endpoint security | 4,600+ techniques, OWASP LLM Top 10, guardrail scoring |
| Binary | Compiled binary and bytecode analysis | Static heuristics and known weakness mapping |
| Network | Host discovery, port scanning, service fingerprinting | CVE-aware service assessment |
| Code Quality | Complexity, duplication, maintainability | Structural analysis and refactor guidance |

Each scanner can run independently or as part of a unified scan profile.

## 4) Platform Runtime Model

### 4.1 Launcher

The user-facing application is a desktop launcher built with Wails (Go backend + embedded web UI). It controls scan workflows locally. There is no browser dependency for runtime operation.

### 4.2 Backend

The backend is a Go 1.24 service using the Gin HTTP framework with:

- **PostgreSQL** (pgx/v5 connection pool: 100 max connections, 15 min connections, 30m lifetime, 30s health checks)
- **Redis** (cache, pub/sub for cross-process WebSocket broadcasting, task queue broker)
- **Asynq** distributed task queue (10 worker goroutines, 5 queue priorities, 30s shutdown drain)

### 4.3 API Surface

The backend exposes approximately **894 HTTP endpoint registrations** organized across public, authenticated, admin, customer, scanner, enterprise, and tool route groups. All authenticated endpoints require JWT Bearer tokens (15-minute expiry, refresh flow, optional TOTP MFA).

### 4.4 Real-Time Updates

Scan progress is broadcast via Redis pub/sub (channel pattern `elite:broadcast:{userID}`) to WebSocket connections. Per-user limit: 15 connections. Global limit: 1,000 connections. Message buffer: 256 per connection.

## 5) SAST Detection Engine (Deep Technical Detail)

The SAST engine is the most complex scanner module. Its detection pipeline has multiple stages:

### 5.1 Rule Loading

- **420 rule sets** (201 local phase-based sets + 219 nativerules package sets)
- Rules compile into a single flat slice cached in memory
- Maximum file size for analysis: **2 MiB** per file
- Concurrency: **8 parallel file workers**

### 5.2 Detection Methods (Per File)

1. **Regex pattern matching:** each rule's compiled pattern is tested against each line. Multi-line context uses a 5-line snippet window (current line ± 2 lines joined).

2. **AST analysis:** language-specific AST analyzers (Go, Python, JavaScript, TypeScript, Java, etc.) perform structure-aware detection beyond regex.

3. **Advanced taint analysis (`AdvancedTaintEngine`):**
   - Extracts function boundaries (regex baseline + optional tree-sitter merge)
   - Traces tainted variables from sources to sinks within functions
   - Maximum call depth: 5. Maximum file size: 200KB.
   - Supports Spring/JAX-RS annotation pre-population (`@RequestParam`, `@PathVariable`, `@RequestBody`, `@RequestHeader`, `@CookieValue`, `@MatrixVariable`) — parameters are automatically marked as tainted
   - Source/sink catalog: **136 sources**, **84 sinks**, **205 sanitizers** across Go, Python, JavaScript, Java, C#, Ruby, PHP, Kotlin, TypeScript, Rust, Swift
   - Vulnerability classes covered: SQLi, command injection, XSS, SSRF, path traversal, open redirect, deserialization, XXE, LDAP injection, NoSQL injection, SSTI, log injection
   - SQLi sanitization heuristic: detects `?`, `$1`, `:1`, `@`, `Prepare`, `parameterize` on sink lines

4. **Direct injection detector:** detects source + sink + concatenation on the same line (method 1) or tracks variable assignments from sources and checks subsequent sink usage within 30 lines (method 2). Docstring-aware masking prevents false matches inside documentation blocks.

5. **Contextual pattern matcher:** searches up to 10–15 non-comment lines for multi-line vulnerability patterns.

### 5.3 False Positive Suppression Pipeline

After a rule matches, the finding passes through these suppression checks (in order):

| # | Check | Purpose |
|---|-------|---------|
| 1 | `rule.Negative` pattern | Rule-defined exclusion pattern |
| 2 | `GetExtractedNegative` | Auto-extracted negative patterns from rule regex |
| 3 | `isCommentLine` | Language-aware comment detection |
| 4 | `isRuleDefinitionLine` | Skips lines that define security rules (MustCompile, NativeRule, assert patterns) |
| 5 | `isDataOrStringLiteral` | Skips struct tags, JSON keys, import/export statements, format strings |
| 6 | `isCmdPackagePath` + `isLibraryOnlyRule` | Skips library-internal rules in cmd packages |
| 7 | `isTestOrBenchmarkPath` | Skips test/benchmark files (directory-boundary aware) |
| 8 | `isGoStdlibFalsePositive` | Rule-ID-specific Go standard library exclusions |
| 9 | Context window validation | ±2/+8 line context check for validation patterns |
| 10 | `isNonCodeFile` | Skips non-code extensions unless rule is infrastructure-specific |
| 11 | Go indented `var` suppression | Skips variable declarations |

### 5.4 Post-Scan Validation

After per-file scanning, findings pass through:

1. **FP Memory:** historical false positive tracking
2. **FP Eliminator:** test file detection, test code detection, comment detection, dead code detection (language-specific: bare `return`, `if false`, `@Deprecated`, `raise NotImplementedError`, etc.), framework-specific suppression
3. **Deep Confidence Scoring:** weighted composite score from 7 factors:
   - Base confidence from rule definition (0.5–0.9)
   - Severity boost (critical +0.15, high +0.10, medium +0.05)
   - Pattern specificity (dangerous substring density, line length penalty)
   - Code context (path heuristics: production 0.9, test 0.3, examples 0.2, vendor 0.4)
   - Historical true-positive rate (requires 10+ fires, else default 0.7)
   - File reputation (requires 5+ findings, else 0.5)
   - Environment factor (production 0.9, test 0.3, default 0.7)
   - Final formula: `Base×0.25 + SeverityBoost + PatternSpecificity×0.15 + CodeContext×0.20 + HistoricalTP×0.15 + FileReputation×0.10 + Environment×0.15`, clamped [0, 1]
   - Drop threshold: findings with score < 0.3 are eliminated
   - Label bands: ≥0.9 very_high, ≥0.7 high, ≥0.5 medium, ≥0.3 low, else very_low
4. **Dynamic severity adjustment**
5. **Low-confidence and INFO-severity stripping**

### 5.5 Limits

- Maximum hits per rule per file: **1**
- Maximum findings per scan: **10,000** (lower-severity findings truncated first)

## 6) SCA Detection Engine

### 6.1 Dependency Parsing

173 registered parsers covering:

- **Core ecosystems:** npm, PyPI, Maven, NuGet, Go modules, RubyGems, Cargo, Composer, CocoaPods, SwiftPM, Conda, Pub, Hex, Conan, vcpkg, CPAN
- **Extended coverage:** OS package managers, cloud/IaC (Terraform, CloudFormation, Pulumi, Ansible Galaxy, Helm, Kustomize, Docker Compose, K8s manifests, Bicep, CDK), mobile, C/C++, Linux distro packages, data/ML, gamedev, web front-end, CI/CD, container/runtime manifests

### 6.2 Vulnerability Intelligence

- **Primary:** OSV (local index batch query, remote API fallback)
- **Secondary:** NVD CVES 2.0 API (when API key configured)
- **Enrichment:** EPSS (Exploit Prediction Scoring System) batch lookup, CISA KEV (Known Exploited Vulnerabilities) batch lookup
- **Reachability analysis:** optional per-package reachability check; unreachable dependencies may have exploitability downgraded

### 6.3 Registry Intelligence

Parallel latest-version lookups for patch-available detection and upgrade guidance.

## 7) IaC Security Engine

### 7.1 Format Support

Terraform (`.tf`, `.tfvars`), Kubernetes YAML (detected by `apiVersion` + `kind`), Dockerfile, CloudFormation (detected by `AWSTemplateFormatVersion` / `Resources:`), Helm, Ansible.

### 7.2 Rule Scale

| Rule Category | Count |
|--------------|-------|
| Base inline rules (TF, K8S, DF, CFN) | 26 |
| Advanced regex rules | ~2,115 |
| Structured HCL/K8s/CFN evaluators | 120 |
| Plan scanner rules | 37 |
| **Total** | **~2,300** |

### 7.3 Detection Methods

- **Regex per-line scanning** with resource-type substring matching
- **Block-aware HCL evaluation** for Terraform attribute analysis
- **Kubernetes RBAC helpers** for permission analysis
- **CIS benchmark alignment** (Azure, GCP rule sets)

## 8) Container Security Engine

- **Image scanning:** Trivy (primary), Grype (fallback), Syft (SBOM/package extraction)
- **Dockerfile analysis:** 80 enhanced rules (critical, high, medium security + best practice + performance)
- **Advanced scanner:** 200+ checks from CIS Docker Benchmark, image security, runtime, network, supply chain, Kubernetes, compliance
- **Built-in fallback:** `builtInDockerScan` when external tools produce no results

## 9) DAST Engine

### 9.1 Check Scale

- **Base scanner:** SQLi, XSS, path traversal, command injection, SSRF active checks
- **Enterprise checks:** 100 checks across 7 categories (Advanced Injection 15, Advanced Authentication 15, API Security 20, Client-Side Advanced 15, Infrastructure 15, Business Logic 10, Compliance & Hardening 10)
- **CVE-specific checks:** 33 CVE-oriented detections
- **Passive scanner:** security headers, information leak, cookies, CORS, SSL/TLS analysis
- **Advanced scanner:** 9 category functions (injection, auth/session, client-side, info disclosure, business logic, protocol, compliance, SSRF, cookie security)

## 10) Proscan Deep (Elite Engine)

Proscan Deep is the platform's most comprehensive dynamic scanner. It is a full in-process security testing engine — not a wrapper around an external tool — implemented across **605 Go source files** under `internal/scanner/elite/`.

### 10.1 Check Catalog

The engine ships with **922 registered security checks** (`proscan_checks_catalog.go`), distributed by execution scope:

| Category | Check Count |
|----------|------------|
| PerServer (server-level assessment) | 108 |
| PerScheme (protocol/scheme-level) | 54 |
| PerDirectory (directory-level) | 46 |
| PerFile (file-level) | 14 |
| PostScan (post-scan validation) | 10 |
| General (parameter/payload-level) | 636 |
| Deepscan (V8/client-side) | 6 |
| **Total** | **922** |

### 10.2 CWE Coverage

Checks are mapped to **30+ distinct CWE identifiers** through the check catalog and the runtime `mapVulnTypeToCWE` enrichment layer:

CWE-22 (Path Traversal), CWE-74 (Injection), CWE-78 (OS Command Injection), CWE-79 (XSS), CWE-89 (SQL Injection), CWE-90 (LDAP Injection), CWE-94 (Code/Template Injection), CWE-98 (PHP Remote File Inclusion), CWE-113 (HTTP Header Injection), CWE-200 (Information Exposure), CWE-235 (Parameter Pollution), CWE-284 (Authorization Bypass), CWE-287 (Authentication Bypass), CWE-295 (Certificate Validation), CWE-310 (Cryptographic Issues), CWE-350 (IP Spoofing), CWE-352 (CSRF), CWE-362 (Race Condition), CWE-434 (Unrestricted File Upload), CWE-444 (HTTP Request Smuggling), CWE-489 (Debug Mode), CWE-502 (Deserialization), CWE-540 (Source Code Exposure), CWE-601 (Open Redirect), CWE-611 (XXE), CWE-639 (IDOR), CWE-643 (XPath Injection), CWE-644 (Improper HTTP Headers), CWE-693 (Protection Mechanism Failure), CWE-918 (SSRF), CWE-942 (Permissive CORS), CWE-943 (NoSQL Injection), CWE-1021 (Clickjacking), CWE-1333 (ReDoS).

### 10.3 Test Groups

The engine organizes payload-based testing into **20 test groups**, each running concurrently per parameter:

| # | Test Group | Tests (AddTest) |
|---|-----------|----------------|
| 1 | SQL Injection | 17 |
| 2 | XSS (Reflected/Stored) | 12 |
| 3 | Path Traversal | 8 |
| 4 | Authentication/Authorization | 16 |
| 5 | CVE-Specific Exploits | 5 |
| 6 | SSRF | 4 |
| 7 | Command Injection | 8 |
| 8 | Deserialization | 3 |
| 9 | Open Redirect / HPP | 2 |
| 10 | File Upload | 2 |
| 11 | CSP / CORS / Cache | 3 |
| 12 | HTTP Smuggling | varies |
| 13 | GraphQL | 5 |
| 14 | Weak Credentials | varies |
| 15 | PHP Configuration | 6 |
| 16 | Header Injection | 1 |
| 17 | LFI (Local File Inclusion) | 1 |
| 18 | Information Disclosure | 8 |
| 19 | DOM XSS (optional, V8) | 3 |
| 20 | Metasploit Exploitation (optional) | 1 |

### 10.4 Scan Lifecycle

1. **Connectivity precheck** — target reachability and response baseline
2. **8-phase precheck sequence:** connectivity, error page profiling, WAF fingerprinting, technology detection, site mapping, parameter discovery, baseline behavior, security baseline
3. **Crawling** — configurable depth (default 5), 60 crawler workers, max 300 pages, 150 directories, 600 files
4. **Parallel pipeline execution** — main target + optional subdomain pipelines
5. **Test group execution** — all 20 groups run concurrently per discovered parameter
6. **Module execution** — per-server (5), per-directory (3), per-file (4), tech-specific (3) dispatcher modules
7. **False positive verification** — evidence-based validation per vulnerability type (SQLi requires SQL error evidence, XSS requires script execution potential, LFI requires file content, RCE requires execution evidence)
8. **CWE and OWASP enrichment** — findings mapped to CWE IDs and OWASP Top 10 2021 categories
9. **Bulk persistence and report generation**

### 10.5 Configuration

| Parameter | Default |
|-----------|---------|
| Crawl depth | 5 |
| Max concurrent requests | 100 (50 via API) |
| Scan timeout | 30 minutes |
| Crawler workers | 60 |
| Max pages to crawl | 300 |
| Max directories | 150 |
| Max files | 600 |
| Max sessions | 10 (30 with init) |
| Session timeout | 30 minutes |
| Max redirects | 10 |
| Max body size | 1 MiB |
| Blind SQLi threshold | 1 second |
| Blind detection timeout | 10 seconds |
| Differential threshold | 0.15 |
| Resource governor | 5,000 goroutines, 1,000 queue |
| RPS limit | 100 |
| Task timeout (Asynq) | 24 hours |

### 10.6 False Positive Controls

The Elite engine applies evidence-based FP prevention at the finding ingestion level:

- **SQL Injection:** requires HIGH confidence, verified flag, or actual SQL error in response — low-confidence SQLi without evidence is dropped
- **XSS:** requires verified script execution potential — payloads reflected only in error messages are dropped
- **LFI/Path Traversal:** requires actual file content in response body (e.g., `root:x:0:0:` for `/etc/passwd`)
- **RCE/Code Execution:** requires verified command execution evidence
- **Mislabel correction:** detects and corrects cross-category mislabeling (e.g., XSS payload incorrectly classified as SSRF)
- **OpenID/OAuth endpoint exclusion:** static configuration endpoints are excluded from SQLi testing

### 10.7 Data Scale

The engine embeds large datasets for payload generation and pattern matching:

- **Proscan checks catalog:** 922 checks
- **Tech-to-vulnerability mappings:** ~65,000 entries
- **Tool-specific mappings:** ~21,000 entries
- **Massive payload loader, CVE loader, advanced payload loader:** runtime-loaded datasets
- **Binary analysis strings:** 138,000+
- **Algorithm strings:** 617,000+

## 11) AI/LLM Security Engine

- **4,600+ attack techniques** across ~90 Go implementation files
- **3 top-level categories:** direct, indirect, evasive
- **50+ subcategories** including: prompt injection, jailbreak, cognitive manipulation, data exfiltration, multimodal attacks, RAG poisoning, agentic workflow exploitation, supply chain, plugin abuse, MCP exploitation, compliance evasion, system prompt extraction
- **Full OWASP LLM Top 10 mapping** (LLM01–LLM10) with per-technique categorization
- **Guardrail effectiveness scoring** for deployed model defenses

## 12) Secrets Detection Engine

- **34 regex patterns** covering: AWS, GCP, Azure, GitHub, Stripe, Slack, SendGrid, Twilio, Mailchimp, Heroku, JWT, auth tokens, private keys, database credentials, passwords, generic secrets
- **Shannon entropy analysis:** parallel pass for high-entropy strings ≥20 characters with configurable threshold (default 4.5)
- **Confidence grading:** entropy > 4.0 → high confidence; else medium; entropy-only detections → low confidence
- **False positive filtering:** placeholder detection, test value filtering, common password exclusion
- **File limits:** 10 MB max file size; binary extensions and common non-code directories skipped

## 13) Cross-Scanner Correlation

The `CrossScannerCorrelator` implements **12 correlation rules** (XCORR-001 through XCORR-012):

| Rule | Scanner Pair | Correlation Logic |
|------|-------------|-------------------|
| XCORR-001 | SAST + SCA | CVE/CWE match |
| XCORR-002 | SAST + Secrets | Same file, lines within 3 |
| XCORR-003 | Binary + SCA | CVE match |
| XCORR-004 | Container + SCA | CVE match |
| XCORR-005 | SAST + API | CWE/keyword class match |
| XCORR-006 | Secrets + Container | Dockerfile path correlation |
| XCORR-007 | IaC + Container | Keyword overlap |
| XCORR-008 | SAST + Binary | CWE match |
| XCORR-009 | IaC + Secrets | Same file / .tf proximity |
| XCORR-010 | Secrets + API | Credential + exposure wording |
| XCORR-011 | Container + Binary | CVE match |
| XCORR-012 | SAST + IaC | SSRF + public / SQLi + public DB |

Correlation only **elevates** confidence and severity — it never suppresses findings.

## 14) Auto-Remediation Engine

- **CWE-keyed fix templates:** regex pattern + replacement pairs per CWE and language
- **Optional LLM-assisted fixes:** configurable model (default GPT-4), 30s timeout, max 3 suggestions per finding
- **Output:** fix suggestion with diff hunks, confidence score, breaking-change flag, and review-required flag
- **Safety controls:** dry-run mode by default, optional git commit integration

## 15) ML-Based Classification

- **False positive classifier:** trained model (JSON weights/bias) with feature extraction; fallback to heuristic scoring when model unavailable
- **Vulnerability prioritizer:** weighted multi-factor scoring (severity, CVSS, exploitability, asset criticality, exposure window, age decay, threat intelligence, business impact, remediation effort, attack complexity)
- **Degraded mode support:** graceful fallback when ML service is unavailable

## 16) Compliance Evidence

### 15.1 Supported Frameworks

OWASP Top 10, PCI DSS 4.0, NIST 800-53, NIST CSF, FISMA, HIPAA, SOC 2, ISO 27001, GDPR, CIS Benchmarks, CERT C, CERT Java, CISA KEV.

### 15.2 Mapping Method

Findings carry CWE identifiers. Each framework control defines a list of related CWEs. The compliance mapping engine matches finding CWEs to control CWE lists and produces:

- Control-level status (violated, at risk, passing)
- Gap analysis (expected CWEs vs. matched CWEs per control)
- Compliance report with evidence linking

### 15.3 Scope Statement

Proscan compliance outputs provide **technical control evidence and coverage visibility**. They do **not** independently certify full framework compliance, which also requires non-technical controls (policies, training, physical security) and auditor judgment.

## 17) Evidence and Reporting

### 16.1 Output Formats

HTML, PDF, JSON, CSV, SARIF, XML.

### 16.2 Report Styles

Comprehensive, Executive, Pentest, Compliance, JSON, DAST.

### 16.3 Visual Themes

Dark, Light, Blue, Terminal.

### 16.4 Scan Coverage Metadata

Reports include per-file detail: files scanned, lines scanned, languages detected, findings per file, rules applied, and scanner selection.

## 17) Operational Security Controls

### 17.1 Middleware Chain (Exact Order)

| # | Middleware | Configuration |
|---|-----------|---------------|
| 1 | Observability | Correlation ID (X-Request-ID), structured logging, path normalization |
| 2 | Metrics | Prometheus HTTP metrics (namespace: proscan) |
| 3 | CORS | Dev: any origin; Prod: explicit origin allowlist |
| 4 | Security Headers | X-Content-Type-Options, X-Frame-Options: DENY, X-XSS-Protection, Referrer-Policy, Permissions-Policy, HSTS (prod only) |
| 5 | Safe Recovery | Panic recovery without stack trace leakage |
| 6 | Brute Force | Progressive delay after 5 failures (1s→2s→4s→8s), lockout at 5/10/15 failures (15m/30m/60m) |
| 7 | Request Size Limit | 10 MiB global body cap |

### 17.2 Rate Limiting

| Tier | Limit | Burst |
|------|-------|-------|
| Auth | 10/min | 5 |
| Public | 30/min | 10 |
| Protected (per-user) | 120/min | 30 |
| Webhooks | 60/min | 20 |
| Scan creation (per-user) | 10/min | 3 |

### 17.3 Resilience

- **Circuit breaker:** per-server (sony/gobreaker), trips at ≥80% failure ratio over 5+ requests, 120s recovery window
- **Degraded mode:** per-feature health tracking for graceful degradation when ML/external services are unavailable
- **DB cache:** in-memory TTL with stale-while-revalidate (5m fresh, 30m stale window, 1m eviction cycle)

## 19) Startup Sequence

1. Anti-debug and system resource checks
2. GOMAXPROCS = max(2, numCPU/2); GOGC = 50
3. Environment loading (.env search)
4. Configuration load and rule embedding
5. Structured logging initialization (slog, JSON format)
6. OpenTelemetry tracing initialization
7. Tool manager (nuclei, ffuf, subfinder download/update)
8. PostgreSQL connection pool + schema migrations + admin bootstrap
9. Background pruners (webhooks, notifications, payments, SCA cleanup)
10. License validator + 60-second license monitor
11. Degraded mode tracker + DB cache
12. Stalled scan monitor (5-minute interval)
13. SaaS scan processor (30-second interval)
14. FFuf and vulnerability sync manager initialization
15. Redis pub/sub broadcast subscriber
16. Asynq worker + scheduler (if enabled)
17. Gin HTTP server (port 8001, 10m read/write timeout, 10s header timeout, 120s idle, 1 MiB max headers)
18. Graceful shutdown (SIGINT/SIGTERM, 5-second drain)

## 20) Limitations

- Automated scanning does not replace manual threat modeling, architecture review, or targeted penetration testing.
- Compliance outputs represent technical evidence and control mapping. They are not legal attestation and do not cover non-technical controls.
- Findings require business-context triage (asset criticality, exploitability, exposure window).
- Taint analysis is intra-file with limited call-depth traversal (max 5). Full inter-procedural and cross-file data flow is not yet implemented.
- DAST testing is target-centric (supplied URL/parameters), not full-spider crawling in all modes.
- ML classifier quality depends on model training data; heuristic fallback is used when the model is unavailable.

## 21) Conclusion

Proscan consolidates twelve scanner modules, layered detection methods, cross-scanner correlation, and audit-ready reporting into a single on-premises platform. The engineering emphasis is on measurable detection quality — through structured confidence scoring, multi-layer false positive suppression, and transparent correlation rules — rather than raw finding volume. The platform is designed for teams that need both engineering-actionable output and compliance-grade evidence without external data dependencies.
