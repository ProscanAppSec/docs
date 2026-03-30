# Detection Methodology

This document describes the algorithms and detection strategies used by Proscan scanners. It provides formal descriptions of how each detection method works without exposing proprietary rule databases or implementation source code.

---

## 1) Detection Method Taxonomy

Proscan does not rely on a single detection technique. Each scanner module combines multiple methods, selected based on target type and vulnerability class:

| Method | Scanner(s) | Description |
|--------|-----------|-------------|
| Pattern/Rule Matching | SAST, Secrets, IaC | Compiled regex patterns tested against source lines |
| AST Analysis | SAST | Language-specific abstract syntax tree traversal |
| Taint Propagation | SAST | Source-to-sink data flow tracking within functions |
| Direct Injection Detection | SAST | Single-line and near-line source+sink+concatenation detection |
| Contextual Pattern Matching | SAST | Multi-line non-comment window analysis |
| Active Payload Testing | DAST, Deep | Request mutation and response differential analysis |
| Entropy Analysis | Secrets | Shannon entropy calculation on candidate strings |
| Dependency Graph Analysis | SCA | Manifest parsing, version resolution, CVE correlation |
| Configuration Policy Checks | IaC, Container | Resource attribute and policy evaluation |
| Protocol Fuzzing | Deep | Multi-strategy payload generation and injection |

## 2) SAST: Multi-Method Static Analysis

### 2.1 Pattern Matching

```
Algorithm: Rule-Based Pattern Matching
──────────────────────────────────────
Input:  File F, RuleSet R (200,000+ compiled rules)
Output: Set of candidate findings

1. Detect language L from file extension (221 extensions → 60 languages)
2. Read file into lines[] (skip if file > 2 MiB)
3. For each rule r ∈ R where r.Languages includes L:
   a. For each line[i] in lines[]:
      i.   If r.Pattern matches line[i]:
           - If r.Negative is defined AND r.Negative matches line[i] → skip
           - If auto-extracted negative matches → skip
           - Build 5-line snippet (lines[i-2..i+2] joined)
           - Emit candidate finding (rule, file, line, snippet)
      ii.  Limit: max 1 hit per rule per file
4. Return all candidate findings
```

Each rule carries:
- A compiled regex pattern (positive match)
- An optional negative pattern (known-safe context suppression)
- Language bindings (which file extensions it applies to)
- CWE mapping, severity, confidence, remediation guidance

### 2.2 Taint Propagation

See [Taint Analysis Model](taint-analysis.md) for the full formal specification.

```
Algorithm: Intra-Procedural Taint Propagation
─────────────────────────────────────────────
Input:  Function F, SourceCatalog S, SinkCatalog K, SanitizerCatalog Z
Output: Set of taint flows {(source_line, sink_line, class, confidence)}

1. Extract function boundaries from file
   - Primary: regex-based extraction
   - Enhancement: tree-sitter AST merge (where available)

2. For each function F:
   a. Initialize tainted_vars = {}
   b. If F has framework annotations (@RequestParam, @PathVariable,
      @RequestBody, @RequestHeader, @CookieValue, @MatrixVariable):
      → Pre-populate all annotated parameters as tainted
   c. For each statement in F.body (top-down):
      - If statement matches source s ∈ S → mark LHS variable as tainted
      - If statement assigns from tainted variable → propagate taint
      - If statement calls function known to return taint → mark LHS
   d. For each statement matching sink k ∈ K:
      - If any argument is a tainted variable:
        i.   Check if argument is a literal → skip (not user-controlled)
        ii.  Check for sanitizer z ∈ Z between source line and sink line
        iii. If no sanitizer found → emit flow
             confidence = 0.75 base, boosted by evidence quality

3. Filter: drop flows where confidence < 0.50
4. Return remaining flows with CWE, severity, remediation
```

### 2.3 Direct Injection Detection

```
Algorithm: Source-Sink Co-occurrence Detection
──────────────────────────────────────────────
Input:  File lines[], InjectionRuleSet IR
Output: Set of injection findings

For each rule ir ∈ IR:
  Method 1 (same-line):
    For each line[i]:
      If ir.source_pattern AND ir.sink_pattern AND ir.concat_pattern
      all match line[i]:
        If ir.negative_pattern does NOT match → emit finding

  Method 2 (variable tracking):
    For each line[i] matching ir.source_pattern:
      Extract assigned variable name via assignment regex
      Store (variable, line_number) in source_vars
    For each subsequent line[j] where (j - i) < 30:
      If ir.sink_pattern AND ir.concat_pattern match line[j]
      AND line[j] contains tracked variable:
        Emit finding with source_line=i, sink_line=j
```

### 2.4 Contextual Pattern Matching

```
Algorithm: Multi-Line Context Analysis
──────────────────────────────────────
Input:  File lines[], ContextRules CR
Output: Set of context-aware findings

For each rule cr ∈ CR:
  Define window W = 10-15 non-comment lines
  For each position p in file:
    Collect W consecutive non-comment lines starting at p
    If cr.primary_pattern matches any line in W
    AND cr.secondary_pattern matches any other line in W:
      If no negative context pattern matches within W:
        Emit finding at primary match position
```

## 3) DAST/Deep: Active Security Testing

### 3.1 Payload Generation Strategy

```
Algorithm: Context-Aware Payload Selection
─────────────────────────────────────────
Input:  Parameter P, injection context C, test group G
Output: Ordered list of payloads to test

1. Classify parameter context:
   - URL parameter, POST body, header, cookie, JSON field, XML element
2. Select base payloads from G's payload database
3. Apply mutation strategies:
   - Encoding permutation (URL, double-URL, Unicode, hex, base64)
   - Case variation
   - Comment insertion (for SQL contexts)
   - Boundary manipulation (for numeric contexts)
   - Null byte injection
   - Polyglot construction
4. Order by: detection probability (highest first), encoding complexity
5. Apply negative filter: skip payloads matching known-safe response patterns
```

### 3.2 Response Differential Analysis

```
Algorithm: Behavioral Differential Detection
────────────────────────────────────────────
Input:  Baseline response B, test response T, payload P
Output: Boolean (vulnerable) + confidence score

1. Compare T against B on:
   - Status code change (e.g., 200 → 500)
   - Response body length differential (threshold: 15% change)
   - Response time differential (threshold: 1 second for time-based blind)
   - Error pattern presence (SQL errors, stack traces, debug output)
   - Payload reflection (for XSS: script execution context)
   - File content presence (for LFI: known file signatures)
   - DNS/HTTP callback receipt (for OOB: out-of-band confirmation)

2. Confidence assignment:
   - OOB confirmation → HIGH (verified exploitable)
   - Error pattern + differential → HIGH
   - Reflection in executable context → HIGH (XSS)
   - Time differential only → MEDIUM (blind)
   - Body length change only → LOW (requires manual verification)

3. Evidence-based validation:
   - SQLi: require SQL error message OR time differential ≥ threshold
   - XSS: require payload in executable HTML context (not error page)
   - LFI: require file content signature in response body
   - RCE: require command output evidence
   - If evidence insufficient → suppress finding (prevent FP)
```

### 3.3 Crawling and Discovery

```
Algorithm: Depth-Limited Attack Surface Discovery
─────────────────────────────────────────────────
Input:  Seed URL, depth limit D (default 5), auth context
Output: Discovered endpoints with parameters

1. Initialize frontier = {seed_url}, visited = {}
2. While frontier is not empty AND depth ≤ D:
   a. Dequeue URL u from frontier
   b. Fetch u with auth context
   c. Parse response for:
      - HTML links (href, action, src)
      - JavaScript endpoints (fetch, XMLHttpRequest, axios patterns)
      - API endpoints (OpenAPI/Swagger discovery)
      - Form inputs and parameters
      - Comments containing paths
   d. For each discovered URL:
      - If not in visited AND within scope → add to frontier
   e. Add u to visited with extracted parameters
3. Return endpoint map with parameter metadata
```

## 4) Secrets Detection

```
Algorithm: Dual-Mode Secret Detection
─────────────────────────────────────
Input:  File F, PatternSet P (34 patterns), entropy threshold T (4.5)
Output: Set of secret findings

Mode 1 — Pattern Matching:
  For each line in F:
    For each pattern p ∈ P:
      If p.regex matches line:
        Extract matched value
        Calculate Shannon entropy of matched value
        If entropy > 4.0 → confidence = HIGH
        Else → confidence = MEDIUM
        Apply FP filter (placeholder, test value, common password check)
        If not filtered → emit finding

Mode 2 — Entropy Analysis (parallel):
  For each line in F:
    Extract quoted strings and assignment values ≥ 20 characters
    Calculate Shannon entropy for each candidate
    If entropy ≥ T (4.5):
      Emit finding with confidence = LOW, type = "high-entropy"
```

## 5) SCA: Dependency Analysis

```
Algorithm: Vulnerability-Enriched Dependency Analysis
────────────────────────────────────────────────────
Input:  Project directory, parser registry (173 parsers)
Output: Dependency graph with vulnerability annotations

1. Walk directory, skip non-relevant paths (.git, node_modules, vendor)
2. For each file, select parser by filename pattern
3. Parse manifests → flat dependency list (name, version, ecosystem, PURL)
4. Resolve transitive dependencies where lockfile provides them
5. Query vulnerability databases:
   a. Primary: OSV local index (batch query)
   b. Fallback: OSV remote API
   c. Secondary: NVD CVES 2.0 API (when API key available)
6. Enrich with exploit intelligence:
   a. EPSS batch lookup (exploit prediction score)
   b. CISA KEV batch lookup (known exploited flag)
7. Optional reachability analysis:
   - If dependency count ≤ threshold: analyze whether vulnerable
     functions are actually called in the codebase
   - Unreachable → downgrade exploitability
8. Registry lookup: check for available patches (latest version)
9. Return enriched dependency graph
```

## 6) IaC: Policy and Configuration Analysis

```
Algorithm: Multi-Layer IaC Security Evaluation
─────────────────────────────────────────────
Input:  IaC files, rule database (~2,300 rules)
Output: Misconfiguration findings

Layer 1 — Regex-Based Detection:
  For each file:
    Detect format (Terraform, K8s, Dockerfile, CloudFormation, Helm, Ansible)
    For each rule matching format:
      If rule.pattern matches line → emit finding

Layer 2 — Structured Evaluation:
  For Terraform: parse HCL blocks → evaluate resource attributes
    against structured rules (e.g., "aws_s3_bucket must not have acl=public-read")
  For Kubernetes: parse YAML → evaluate pod spec, RBAC, network policies
  For Dockerfile: parse instructions → evaluate against best practices
  For CloudFormation: parse resources → evaluate security configuration

Layer 3 — Plan Analysis:
  For Terraform plan output: evaluate planned changes against
  37 plan-specific rules to catch drift before apply
```
