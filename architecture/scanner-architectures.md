# Scanner Architecture Diagrams

This document provides architecture-level diagrams and technical internals for each Proscan scanner module. All numbers and configuration values are from the production codebase.

---

## Common Scanner Contract

All scanners produce output conforming to a shared normalized schema. After scanner-specific detection, findings enter the cross-scanner merge pipeline:

```mermaid
flowchart LR
    I[Scanner-Specific Input] --> P[Preprocessing and Scope Resolution]
    P --> D[Detection Core]
    D --> V[Scanner-Specific Validation]
    V --> N[Schema Normalization<br/>CWE, severity, location, evidence]
    N --> X[Cross-Scanner Correlation<br/>12 XCORR rules]
    X --> C[Confidence Harmonization]
    C --> DD[Deduplication<br/>CWE + file + line-group buckets]
    DD --> R[Unified Findings + Reports]
```

---

## 1) SAST — Static Application Security Testing

### Architecture

```mermaid
flowchart TB
    Source[Source Files<br/>221 supported extensions] --> FileWorkers[8 Concurrent File Workers<br/>2 MiB max file size]

    FileWorkers --> RuleEngine[Rule Engine<br/>420 rule sets<br/>200,000+ compiled rules]

    subgraph Detection[Multi-Method Detection]
      Regex[Regex Pattern Matching<br/>per-line + 5-line snippet join]
      AST[AST Analysis<br/>per-language analyzers]
      Taint[Advanced Taint Engine<br/>136 sources / 84 sinks / 205 sanitizers<br/>max call depth: 5]
      Direct[Direct Injection Detector<br/>same-line + 30-line variable tracking]
      Context[Contextual Pattern Matcher<br/>10-15 line non-comment window]
    end

    RuleEngine --> Detection

    Detection --> Suppression[11-Stage FP Suppression Pipeline]

    subgraph Suppression[FP Suppression Pipeline]
      S1[1. Rule negative pattern]
      S2[2. Auto-extracted negatives]
      S3[3. Comment line detection]
      S4[4. Rule definition line detection]
      S5[5. Data/string literal detection]
      S6[6. Library-only rule filtering]
      S7[7. Test/benchmark path filtering]
      S8[8. Go stdlib FP exclusions]
      S9[9. Context window validation]
      S10[10. Non-code file filtering]
      S11[11. Go var declaration filtering]
    end

    Suppression --> PostScan[Post-Scan Validation]

    subgraph PostScan[Post-Scan Validation]
      FPMem[FP Memory<br/>historical tracking]
      FPElim[FP Eliminator<br/>test file, test code, comment,<br/>dead code, framework suppression]
      Conf[Deep Confidence Scorer<br/>7-factor weighted composite<br/>drop threshold: 0.3]
      DynSev[Dynamic Severity Adjustment]
      Strip[Low-confidence / INFO stripping]
    end

    PostScan --> Findings[SAST Findings<br/>max 1 hit per rule per file<br/>max 10,000 per scan]
```

### Taint Analysis Detail

```mermaid
flowchart LR
    FuncExtract[Function Boundary Extraction<br/>regex + tree-sitter merge] --> ParamAnalysis[Parameter Assignment<br/>and Return Propagation]
    ParamAnalysis --> SpringAnnot[Spring Annotation Pre-population<br/>RequestParam, PathVariable,<br/>RequestBody, RequestHeader,<br/>CookieValue, MatrixVariable]
    SpringAnnot --> SourceMatch[Source Matching<br/>.method and annotation patterns]
    SourceMatch --> VarTrack[Tainted Variable Tracking]
    VarTrack --> SinkMatch[Sink Matching<br/>with literal argument exclusion]
    SinkMatch --> SanitCheck[Sanitizer Check Between<br/>Source and Sink Lines]
    SanitCheck --> ConfCalc[Confidence Calculation<br/>0.75 - 1.0 with boosts]
    ConfCalc --> TaintFindings[Taint Flow Findings<br/>drop if confidence < 0.50]
```

**Language coverage for taint catalog:** Go, Python, JavaScript, Java, C#, Ruby, PHP, Kotlin, TypeScript, Rust, Swift.

**Vulnerability classes tracked:** SQL injection, command injection, XSS, SSRF, path traversal, open redirect, deserialization, XXE, LDAP injection, NoSQL injection, SSTI, log injection.

### Confidence Scorer Formula

```
FinalScore = BaseConfidence × 0.25
           + SeverityBoost
           + PatternSpecificity × 0.15
           + CodeContext × 0.20
           + HistoricalTPRate × 0.15
           + FileReputation × 0.10
           + EnvironmentFactor × 0.15
```

Clamped to [0, 1]. Findings below 0.3 are dropped.

| Factor | Range | Notes |
|--------|-------|-------|
| BaseConfidence | 0.5 – 0.9 | From rule definition |
| SeverityBoost | 0 – 0.15 | Critical +0.15, high +0.10, medium +0.05 |
| PatternSpecificity | 0.1 – 1.0 | Dangerous substring density; −0.2 if line < 20 chars |
| CodeContext | 0.1 – 1.0 | Path heuristics (todo/mock/vendor/generated) |
| HistoricalTPRate | 0.0 – 1.0 | Requires 10+ fires; else 0.7 |
| FileReputation | 0.0 – 1.0 | Requires 5+ findings; else 0.5 |
| EnvironmentFactor | 0.2 – 0.9 | Production 0.9, test 0.3, examples 0.2, vendor 0.4 |

---

## 2) DAST — Dynamic Application Security Testing

### Architecture

```mermaid
flowchart TB
    Target[Target URL + Session Context] --> Base[Base Scanner<br/>SQLi, XSS, path traversal,<br/>command injection, SSRF]

    Target --> Enterprise[Enterprise Scanner<br/>100 checks across 7 categories]

    subgraph Enterprise[Enterprise Check Categories]
      E1[Advanced Injection — 15 checks]
      E2[Advanced Authentication — 15 checks]
      E3[API Security — 20 checks]
      E4[Client-Side Advanced — 15 checks]
      E5[Infrastructure — 15 checks]
      E6[Business Logic — 10 checks]
      E7[Compliance and Hardening — 10 checks]
    end

    Target --> CVE[CVE-Specific Checks<br/>33 CVE-oriented detections]

    Target --> Advanced[Advanced Scanner<br/>9 category functions:<br/>injection, auth/session,<br/>client-side, info disclosure,<br/>business logic, protocol,<br/>compliance, SSRF, cookies]

    Target --> Passive[Passive Scanner<br/>security headers, info leak,<br/>cookies, CORS, SSL/TLS]

    Base --> Merge[Result Merge]
    Enterprise --> Merge
    CVE --> Merge
    Advanced --> Merge
    Passive --> Merge
    Merge --> DASTFindings[DAST Findings]
```

### Check Scale Summary

| Layer | Check Count |
|-------|------------|
| Base active checks | 5 vulnerability types |
| Enterprise checks | 100 (7 categories) |
| CVE-specific | 33 |
| Advanced category functions | 9 |
| Passive analysis | Security headers, cookies, CORS, SSL/TLS |
| **Total** | **150+** |

---

## 3) Proscan Deep (Elite Engine)

The largest scanner module: **605 Go source files**, **922 registered security checks**, **30+ CWE categories**, **20 concurrent test groups**.

### Full Lifecycle Architecture

```mermaid
flowchart TB
    Target[Target URL] --> Prechecks[8-Phase Precheck Sequence<br/>connectivity, error pages, WAF,<br/>tech detection, site mapping,<br/>parameters, baseline, security baseline]

    Prechecks --> Crawler[Crawler Engine<br/>60 workers, depth 5,<br/>max 300 pages / 150 dirs / 600 files]

    Crawler --> Pipeline[Parallel Pipeline Execution<br/>main target + subdomain pipelines]

    Pipeline --> TestGroups[20 Test Groups<br/>concurrent per parameter]

    subgraph TestGroups[Test Group Engine]
      TG1[SQLi — 17 tests]
      TG2[Auth — 16 tests]
      TG3[XSS — 12 tests]
      TG4[Path Traversal — 8]
      TG5[CmdI — 8]
      TG6[Info Disclosure — 8]
      TG7[PHP Config — 6]
      TG8[CVE Exploits — 5]
      TG9[GraphQL — 5]
      TG10[SSRF — 4]
      TG11[Deserialization — 3]
      TG12[CSP/CORS/Cache — 3]
      TG13[DOM XSS — 3]
      TG14[Open Redirect/HPP — 2]
      TG15[File Upload — 2]
      TG16[HTTP Smuggling]
      TG17[Weak Credentials]
      TG18[Header Injection — 1]
      TG19[LFI — 1]
      TG20[Metasploit — 1]
    end

    Pipeline --> Modules[Module Dispatcher<br/>5 per-server + 3 per-directory +<br/>4 per-file + 3 tech-specific]

    TestGroups --> FPVerification[Evidence-Based FP Verification<br/>SQLi: SQL error required<br/>XSS: script execution required<br/>LFI: file content required<br/>RCE: execution evidence required]

    Modules --> FPVerification

    FPVerification --> CWEMap[CWE + OWASP Enrichment<br/>30+ CWE IDs, OWASP A01-A10]

    CWEMap --> Findings[Elite Findings<br/>922 check types]
```

### CWE Coverage (30+ Categories)

CWE-22, CWE-74, CWE-78, CWE-79, CWE-89, CWE-90, CWE-94, CWE-98, CWE-113, CWE-200, CWE-235, CWE-284, CWE-287, CWE-295, CWE-310, CWE-350, CWE-352, CWE-362, CWE-434, CWE-444, CWE-489, CWE-502, CWE-540, CWE-601, CWE-611, CWE-639, CWE-643, CWE-644, CWE-693, CWE-918, CWE-942, CWE-943, CWE-1021, CWE-1333.

### Embedded Data Scale

| Dataset | Scale |
|---------|-------|
| Proscan checks catalog | 922 checks |
| Tech-to-vulnerability mappings | ~65,000 |
| Tool-specific mappings | ~21,000 |
| Binary analysis strings | 138,000+ |
| Algorithm strings | 617,000+ |

### Configuration Defaults

| Parameter | Value |
|-----------|-------|
| Crawl depth | 5 |
| Max concurrent | 100 |
| Crawler workers | 60 |
| Max pages / dirs / files | 300 / 150 / 600 |
| Scan timeout | 30 minutes |
| RPS limit | 100 |
| Blind SQLi threshold | 1 second |
| Differential threshold | 0.15 |
| Resource governor | 5,000 goroutines |
| Asynq task timeout | 24 hours |

---

## 4) SCA — Software Composition Analysis

### Architecture

```mermaid
flowchart TB
    Manifests[Dependency Manifests<br/>and Lockfiles] --> Parser[173 Registered Parsers<br/>16 ecosystems + extended coverage]

    subgraph Ecosystems[Supported Ecosystems]
      E1[npm, PyPI, Maven, NuGet,<br/>Go, RubyGems, Cargo, Composer]
      E2[CocoaPods, SwiftPM, Conda,<br/>Pub, Hex, Conan, vcpkg, CPAN]
      E3[Extended: Terraform, CFN, Pulumi,<br/>Ansible Galaxy, Helm, Kustomize,<br/>Docker Compose, K8s, Bicep, CDK,<br/>OS packages, CI/CD, mobile, C/C++,<br/>data/ML, gamedev, container runtime]
    end

    Parser --> DepGraph[Dependency Graph Resolution<br/>direct + transitive + PURL]

    DepGraph --> VulnLookup[Vulnerability Lookup]

    subgraph VulnLookup[Vulnerability Intelligence]
      OSV[OSV<br/>local index batch query<br/>+ remote API fallback]
      NVD[NVD CVES 2.0 API<br/>when API key set]
    end

    VulnLookup --> Enrichment[Enrichment]

    subgraph Enrichment
      EPSS[EPSS Batch Lookup<br/>exploit prediction scoring]
      KEV[CISA KEV Batch Lookup<br/>known exploited vulnerabilities]
    end

    Enrichment --> Reachability[Reachability Analysis<br/>optional per-package check<br/>unreachable → exploitability downgrade]

    Reachability --> Registry[Registry Lookups<br/>latest version detection<br/>patch-available guidance]

    Registry --> SCAFindings[SCA Findings]
```

---

## 5) Secrets Detection

### Architecture

```mermaid
flowchart TB
    Files[Source and Config Files<br/>max 10 MB per file<br/>binary extensions skipped] --> PatternScan[Pattern Scanner<br/>34 compiled regex rules]

    subgraph PatternScan[Pattern Detection]
      AWS[AWS access/secret keys]
      GCP[GCP service account keys]
      Azure[Azure storage/connection strings]
      GitHub2[GitHub tokens]
      Stripe2[Stripe API keys]
      Slack2[Slack tokens/webhooks]
      SendGrid2[SendGrid API keys]
      Twilio2[Twilio credentials]
      More[Mailchimp, Heroku, JWT,<br/>private keys, database URIs,<br/>passwords, generic secrets]
    end

    Files --> EntropyScan[Entropy Scanner<br/>parallel pass]

    subgraph EntropyScan[Entropy Detection]
      Quoted[Quoted string extraction<br/>regex: entropyQuotedRe]
      Assign[Assignment extraction<br/>regex: entropyAssignRe]
      Threshold[Shannon entropy threshold: 4.5<br/>minimum string length: 20 chars]
    end

    PatternScan --> ConfGrade[Confidence Grading<br/>entropy > 4.0 → high<br/>else → medium]
    EntropyScan --> ConfGrade2[Entropy-only → low confidence]

    ConfGrade --> FPFilter[False Positive Filter<br/>placeholder detection<br/>test value filtering<br/>common password exclusion]
    ConfGrade2 --> FPFilter

    FPFilter --> SecretFindings[Secrets Findings]
```

---

## 6) IaC Security

### Architecture

```mermaid
flowchart TB
    Input[IaC Files] --> FormatDetect[Format Detection]

    subgraph FormatDetect[Supported Formats]
      TF[Terraform: .tf, .tfvars]
      K8S[Kubernetes: YAML with apiVersion+kind]
      Docker[Dockerfile: Dockerfile.*]
      CFN[CloudFormation: AWSTemplateFormatVersion]
      Helm2[Helm: YAML charts]
      Ansible2[Ansible: playbooks]
    end

    FormatDetect --> RuleEngine2[Rule Engine]

    subgraph RuleEngine2[Detection Layers — ~2,300 Total Rules]
      Base2[Base Inline Rules — 26<br/>TF001-TF008, K8S001-K8S008,<br/>DF001-DF006, CFN001-CFN004]
      Advanced2[Advanced Regex Rules — ~2,115<br/>AWS 250 + Azure 250 + GCP 250 +<br/>K8s 250 + CFN 200 + Docker 130 +<br/>Compliance 150 + CIS Azure 70 +<br/>CIS GCP 65 + Advanced 500]
      Structured[Structured Evaluators — 120<br/>HCL block-aware, K8s RBAC,<br/>Dockerfile instruction, CFN resource]
      Plan[Plan Scanner — 37 rules<br/>Terraform plan output analysis]
    end

    RuleEngine2 --> IaCFindings[IaC Findings]
```

---

## 7) Container Security

### Architecture

```mermaid
flowchart TB
    Image[Container Image or Dockerfile] --> Primary[Primary: Trivy<br/>vulnerabilities + misconfig]
    Image --> Fallback[Fallback: Grype<br/>if Trivy yields no vulns]
    Image --> SBOM[SBOM: Syft<br/>package extraction]
    Image --> BuiltIn[Built-in Fallback<br/>builtInDockerScan]

    Image --> DockerAnalyzer[Enhanced Dockerfile Analyzer<br/>80 rules across 5 categories]

    subgraph DockerAnalyzer[Dockerfile Rule Categories]
      Crit[Critical Security Rules<br/>secrets in ENV/ARG, SSH keys]
      High2[High Security Rules<br/>curl pipe bash, chmod 777]
      Med[Medium Security Rules]
      Best[Best Practice Rules]
      Perf[Performance Rules]
    end

    Image --> AdvContainer[Advanced Container Scanner<br/>200+ checks]

    subgraph AdvContainer[CIS Docker Benchmark Alignment]
      CIS1[Image security checks]
      CIS2[Runtime configuration checks]
      CIS3[Network security checks]
      CIS4[Supply chain checks]
      CIS5[Kubernetes checks]
      CIS6[Compliance checks]
    end

    Primary --> Merge2[Result Merge + Dedup]
    Fallback --> Merge2
    SBOM --> Merge2
    BuiltIn --> Merge2
    DockerAnalyzer --> Merge2
    AdvContainer --> Merge2
    Merge2 --> ContainerFindings[Container Findings<br/>by severity: critical/high/medium/low<br/>fixable count]
```

---

## 8) API Security

```mermaid
flowchart LR
    Spec[OpenAPI / GraphQL /<br/>Endpoint Definitions] --> SchemaAnalysis[Schema and Route Analysis]
    SchemaAnalysis --> AuthTest[Authentication and<br/>Authorization Testing]
    AuthTest --> InjectionTest[Injection and<br/>Logic Abuse Testing]
    InjectionTest --> BehaviorCheck[Response Behavior Validation]
    BehaviorCheck --> APIFindings[API Findings]
```

API security testing is schema-driven: endpoints are tested for authentication bypass, authorization flaws, injection vulnerabilities, and business logic abuse based on the API specification.

---

## 9) AI/LLM Security

### Architecture

```mermaid
flowchart TB
    Target2[Model Endpoint or<br/>Prompt Surface] --> Registry2[Technique Registry<br/>4,600+ techniques<br/>~90 Go implementation files]

    subgraph Registry2[Technique Categories]
      Direct[Direct: overt, cognitive,<br/>reformulation, boundary, jailbreak]
      Indirect[Indirect: data, document,<br/>web, multimodal]
      Evasive[Evasive: obfuscation, encoding,<br/>multi-step, agent, memory]
      V2[V2: RAG, agentic, recon,<br/>compliance, temporal, format, API]
      V3[V3: privacy, adversarial-ML,<br/>infrastructure, architecture,<br/>training, reasoning, domain]
      V4[V4: supply-chain, plugin,<br/>excessive-agency, output-handling,<br/>overreliance, MCP, RAG,<br/>agentic-workflow, compliance-evasion,<br/>system-prompt]
    end

    Registry2 --> Execution[Technique Execution<br/>against target model]
    Execution --> OutputAnalysis[Output Safety and<br/>Leakage Analysis]
    OutputAnalysis --> GuardrailEval[Guardrail Effectiveness<br/>Evaluation and Scoring]
    GuardrailEval --> OWASPMap[OWASP LLM Top 10 Mapping<br/>LLM01 through LLM10]
    OWASPMap --> AIFindings[AI/LLM Findings<br/>with cost estimation per provider]
```

---

## 10) Binary Analysis

```mermaid
flowchart LR
    Artifact[Compiled Binary or<br/>Bytecode Artifact] --> MetadataExtract[Metadata and<br/>String Extraction]
    MetadataExtract --> StaticAnalysis[Static Binary<br/>Pattern Analysis]
    StaticAnalysis --> WeaknessMap[Known Weakness<br/>Mapping]
    WeaknessMap --> BinaryFindings[Binary Findings]
```

---

## 11) Network Scanner

```mermaid
flowchart LR
    Scope2[CIDR / Host Scope] --> Discovery[Host Discovery]
    Discovery --> PortEnum[Port and Service Enumeration]
    PortEnum --> Fingerprint[Service Fingerprinting]
    Fingerprint --> VulnChecks[CVE-Aware<br/>Vulnerability Checks]
    VulnChecks --> NetFindings[Network Findings]
```

---

## 12) Code Quality

```mermaid
flowchart LR
    Codebase[Source Repository] --> StructAnalysis[Structural Analysis]
    StructAnalysis --> ComplexityCheck[Complexity and<br/>Duplication Checks]
    ComplexityCheck --> MaintScore[Maintainability Scoring]
    MaintScore --> QualFindings[Code Quality Findings]
```

---

## Cross-Scanner Correlation Engine

```mermaid
flowchart TB
    AllOutputs[All Scanner Outputs] --> Normalize2[Schema Normalization<br/>CWE + severity + location]
    Normalize2 --> Pair[Scanner Pair Matching<br/>12 XCORR rules]

    subgraph Pair[Correlation Rules]
      X1[XCORR-001: SAST+SCA — CVE/CWE match]
      X2[XCORR-002: SAST+Secrets — same file, ±3 lines]
      X3[XCORR-003: Binary+SCA — CVE match]
      X4[XCORR-004: Container+SCA — CVE match]
      X5[XCORR-005: SAST+API — CWE/keyword match]
      X6[XCORR-006: Secrets+Container — Dockerfile path]
      X7[XCORR-007: IaC+Container — keyword overlap]
      X8[XCORR-008: SAST+Binary — CWE match]
      X9[XCORR-009: IaC+Secrets — same file / .tf proximity]
      X10[XCORR-010: Secrets+API — credential+exposure]
      X11[XCORR-011: Container+Binary — CVE match]
      X12[XCORR-012: SAST+IaC — SSRF+public / SQLi+public DB]
    end

    Pair --> Boost[Confidence and Severity Elevation<br/>correlation only raises, never suppresses]
    Boost --> Tag[Tag: cross-scanner-correlated<br/>metadata: cross_scanner_boost,<br/>cross_scanner_correlations]
    Tag --> Final[Unified Prioritized Findings]
```

### Correlation Design Principles

- Match by target identity (repository, image, endpoint, host)
- Match by weakness class (CWE and scanner-specific category)
- Preserve distinct findings by location grouping (20-line buckets) to prevent over-collapse
- Correlation only elevates confidence — it never suppresses or removes findings
- Correlated findings are tagged with metadata for audit traceability
