# False Positive Engineering

This document describes how Proscan systematically eliminates false positives while preserving true positive detection. It covers the multi-stage suppression pipeline, post-scan validation, and the design principles behind each stage.

---

## 1) Design Goal

**Target: near-zero false positive rate with zero true positive suppression.**

Every suppression stage must satisfy two invariants:
- It must provably reduce false positives (measured against ground-truth test suites)
- It must not suppress any known true positive (validated against deliberately vulnerable applications)

## 2) SAST False Positive Suppression Pipeline

After a rule pattern matches a source line, the finding passes through an **11-stage suppression pipeline** before reaching post-scan validation. Each stage either passes the finding through or suppresses it with a documented reason.

```
Raw Pattern Match
  │
  ├─ Stage 1: Rule Negative Pattern
  │   Does the rule's own negative pattern match this line?
  │   Purpose: rule-defined safe context (e.g., parameterized query pattern)
  │
  ├─ Stage 2: Auto-Extracted Negative
  │   Does the rule's regex structure imply a safe alternative?
  │   Purpose: automatically derived exclusion from pattern structure
  │
  ├─ Stage 3: Comment Detection
  │   Is this line a comment in the detected language?
  │   Purpose: prevent matching vulnerability patterns inside code comments
  │   Method: language-aware comment syntax detection
  │
  ├─ Stage 4: Rule Definition Detection
  │   Is this line defining a security rule or test assertion?
  │   Triggers: MustCompile, NativeRule{, assert patterns, CWE strings
  │   Purpose: prevent self-detection (scanner finding its own rules)
  │   Scope: only applies to scanner-internal file paths
  │
  ├─ Stage 5: Data Literal Detection
  │   Is this line a struct tag, JSON key, format string, or import statement?
  │   Purpose: prevent matching data definitions that look like code patterns
  │   Exception: credential-containing data literals are NOT suppressed
  │
  ├─ Stage 6: Library-Internal Filtering
  │   Is this a library-only rule matching in a cmd/ package?
  │   Purpose: prevent library usage rules from firing on non-library code
  │
  ├─ Stage 7: Test/Benchmark Path Filtering
  │   Is this file in a test or benchmark directory?
  │   Method: directory-boundary aware matching (not substring)
  │   Purpose: test code intentionally contains vulnerable patterns
  │   Protection: com.example and similar package paths are NOT filtered
  │
  ├─ Stage 8: Standard Library FP Exclusion
  │   Is this a known false positive in Go standard library code?
  │   Method: rule-ID-specific exclusion for standard library patterns
  │   Purpose: stdlib implementations contain patterns that look vulnerable
  │            but are actually safe in context
  │
  ├─ Stage 9: Context Window Validation
  │   Do the ±2 to +8 surrounding lines contain validation patterns?
  │   Purpose: if nearby code validates/sanitizes the input, suppress
  │
  ├─ Stage 10: Non-Code File Filtering
  │   Is this a non-code file (documentation, data, config)?
  │   Exception: infrastructure rules and secret/credential rules
  │              are NOT filtered on config files
  │
  └─ Stage 11: Declaration Filtering
      Is this a variable declaration (not usage)?
      Purpose: declarations define variables but don't execute vulnerable logic
```

### Stage Ordering Rationale

Stages are ordered from cheapest to most expensive:
- Stages 1-2 (pattern matching) are regex tests — O(1) per finding
- Stages 3-6 (line analysis) are string operations — O(line length)
- Stages 7-8 (path analysis) are path matching — O(path length)
- Stages 9 (context window) reads surrounding lines — O(window size)
- Stages 10-11 (classification) are fast lookups

This ordering ensures most false positives are eliminated before expensive checks run.

## 3) Post-Scan Validation Pipeline

After all files are scanned, the accumulated findings pass through four additional validation layers:

### 3.1 FP Memory

Historical tracking of previously identified false positives. If a finding matches a pattern that has been consistently marked as FP across scans, it receives a reduced confidence score.

### 3.2 FP Eliminator

Structural analysis of each finding's code context:

| Check | What It Detects | Why It's Safe to Suppress |
|-------|----------------|--------------------------|
| Test file detection | Files in test directories or with test naming conventions | Test code intentionally contains vulnerable patterns |
| Test code detection | Assertions, test setup, mock patterns within any file | Test constructs are not production code paths |
| Comment detection | Finding line is inside a block or doc comment | Comments describe but don't execute vulnerable code |
| Dead code detection | Code after unconditional return, inside `if false`, `@Deprecated`, `raise NotImplementedError` | Dead code is not reachable at runtime |
| Framework suppression | Framework-specific safe patterns (e.g., ORM parameterization) | Framework provides built-in protection |

### 3.3 Confidence Scoring

See [Confidence Model](confidence-model.md) for the full specification. Findings with a composite score below **0.3** are eliminated. The scorer uses 7 weighted factors to estimate the probability that a finding is a true positive.

### 3.4 Final Filters

- **Low-confidence stripping:** findings with confidence label "very_low" are removed
- **INFO severity stripping:** informational findings are removed from actionable output
- **Dynamic severity adjustment:** severity may be adjusted based on code context

## 4) DAST / Proscan Deep FP Prevention

The Deep scanner applies **evidence-based verification** at the finding ingestion level. Unlike SAST (which uses pattern and context analysis), DAST requires **runtime evidence** that the vulnerability is real:

| Vulnerability Type | Required Evidence | What Gets Suppressed |
|-------------------|-------------------|---------------------|
| SQL Injection | SQL error in response body, OR time-based differential ≥ 1s, OR OOB callback | Low-confidence SQLi without definitive response evidence |
| XSS | Payload appears in executable HTML context (not error page) | Payloads reflected only inside error messages or non-executable positions |
| Path Traversal / LFI | File content signature in response (e.g., `root:x:0:0:` for /etc/passwd) | 200 responses to traversal payloads without actual file content |
| RCE / Code Execution | Command output evidence in response | Reflected payloads without execution proof |
| Mislabeled findings | Automatic cross-category correction | XSS payload incorrectly classified as SSRF, Command Injection, or XXE |
| OpenID/OAuth endpoints | Static JSON configuration endpoints excluded from SQLi testing | False positives from parameter names in OpenID discovery documents |

### Evidence Quality Hierarchy

```
VERIFIED (highest confidence):
  └─ Out-of-band callback received (DNS/HTTP)
  └─ Confirmed exploitable with secondary validation

HIGH:
  └─ SQL error + response differential
  └─ Payload in executable context (script tags, event handlers)
  └─ File content signature present
  └─ Command output pattern matched

MEDIUM:
  └─ Time-based differential only (blind detection)
  └─ Status code change with error pattern

LOW (suppressed by default):
  └─ Response length change only
  └─ Reflection without executable context
  └─ No corroborating evidence
```

## 5) Cross-Scanner FP Reduction

When multiple scanners produce findings for the same target, the cross-scanner correlator can **elevate** confidence (findings confirmed by 2+ scanners are more likely real) but **never suppresses** findings. This is a deliberate design choice — correlation only raises confidence, never lowers it.

## 6) Deduplication vs. Suppression

Deduplication is distinct from FP suppression:

- **Deduplication** merges findings that describe the **same vulnerability** in the **same code location** (within 20-line buckets). This reduces noise without losing coverage.
- **FP suppression** removes findings that are determined to be **not real vulnerabilities**. This reduces false positives.

The dedup key is `(CWE, file, line_group)` — using 20-line buckets to prevent collapsing distinct vulnerabilities in different code regions while still merging duplicate reports of the same issue.

## 7) Validation Methodology

To validate that the FP pipeline does not suppress true positives:

1. **Ground truth test suites:** Run against OWASP Benchmark (2,740 labeled test cases), Juliet Test Suite (80,000+ cases), and deliberately vulnerable applications (WebGoat, DVWA, Juice Shop)
2. **Per-stage measurement:** Each stage is instrumented to count how many findings it suppresses and why
3. **TP suppression audit:** After pipeline completion, compare remaining findings against known vulnerabilities in the test suite. Any true positive that was suppressed indicates a pipeline defect
4. **Regression testing:** New suppression rules are tested against the full ground-truth corpus before deployment

## 8) Metrics Definition

| Metric | Formula | Target |
|--------|---------|--------|
| False Positive Rate | FP / (TP + FP) | < 5% |
| True Positive Suppression Rate | Suppressed TP / Total TP | 0% |
| Pipeline Suppression Ratio | Suppressed / Raw Matches | Informational (higher = more noise removed) |
| Per-Stage Contribution | Stage Suppressions / Total Suppressions | Informational (identifies which stages do the most work) |
