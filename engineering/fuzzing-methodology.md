# Fuzzing and Payload Methodology

This document describes the payload generation and mutation strategies used by Proscan's dynamic scanners (DAST and Proscan Deep). It covers mutation algorithms, payload categories, and detection logic without exposing the full payload database.

---

## 1) Payload Architecture

Proscan Deep maintains a multi-layer payload system:

| Layer | Scale | Purpose |
|-------|-------|---------|
| Check catalog | 922 registered checks | Structured check definitions with CWE mapping |
| Tech-to-vulnerability mappings | ~65,000 entries | Technology-specific payload selection |
| Tool-specific mappings | ~21,000 entries | Integration with extended detection datasets |
| Binary analysis strings | 138,000+ entries | Binary pattern recognition |
| Algorithm strings | 617,000+ entries | Cryptographic and algorithmic analysis |

Payloads are not hardcoded lists — they are generated and mutated at scan time based on the target's technology stack, parameter context, and response behavior.

## 2) Mutation Strategies

### 2.1 Encoding Permutation

Each base payload is tested with multiple encoding schemes to bypass input filters:

```
Strategy: Encoding Permutation
─────────────────────────────
Base payload P → generate variants:
  1. Raw (no encoding)
  2. URL encoding (%xx)
  3. Double URL encoding (%25xx)
  4. Unicode encoding (\uXXXX)
  5. HTML entity encoding (&lt; &#60;)
  6. Hex encoding (0x...)
  7. Base64 encoding
  8. Mixed encoding (combine 2+ schemes)

Selection: test most likely encodings first based on
parameter context (URL param → URL encoding priority,
JSON body → Unicode priority, XML → entity priority)
```

### 2.2 Context-Aware Injection

Payload selection depends on where the parameter appears:

```
Strategy: Context Classification
────────────────────────────────
1. Identify injection context:
   - HTML body → XSS payloads (tag injection, event handlers)
   - HTML attribute → attribute escape payloads
   - JavaScript string → JS escape and injection
   - SQL query → SQL-specific payloads
   - OS command → shell metacharacter payloads
   - URL → redirect and SSRF payloads
   - XML → XXE and entity payloads
   - JSON → structure manipulation
   - HTTP header → header injection payloads

2. Select base payloads from matching context category
3. Apply encoding mutations appropriate to the context
4. Order by detection probability
```

### 2.3 Boundary Value Analysis

```
Strategy: Boundary Manipulation
──────────────────────────────
For numeric parameters:
  - Minimum/maximum integer values
  - Negative values, zero
  - Floating point edge cases
  - Type confusion (string where number expected)

For string parameters:
  - Empty string
  - Very long strings (buffer boundary)
  - Null bytes (%00)
  - Unicode boundary characters
  - Format string specifiers (%s, %x, %n)
```

### 2.4 Polyglot Construction

```
Strategy: Polyglot Payloads
──────────────────────────
Construct payloads that are valid in multiple contexts simultaneously:
  - SQL + XSS polyglots (trigger in either SQL or HTML context)
  - Command + path traversal polyglots
  - XML + XXE + SSRF chain payloads

Purpose: maximize detection probability when the exact
injection context is uncertain
```

### 2.5 Time-Based Differential

```
Strategy: Blind Detection via Timing
────────────────────────────────────
1. Establish baseline response time (median of 3 requests)
2. Send time-delay payload:
   - SQL: SLEEP(N), WAITFOR DELAY, pg_sleep(N)
   - Command: sleep N, ping -c N, timeout /T N
   - XXE: external entity with slow-responding server
3. Measure response time differential
4. Threshold: ≥ 1 second above baseline → candidate finding
5. Confirm with second attempt at different delay value
6. If confirmed → emit finding with MEDIUM confidence (blind)
```

### 2.6 Out-of-Band Detection

```
Strategy: OOB Callback Verification
───────────────────────────────────
1. Generate unique callback identifier per test
2. Construct payload that triggers external callback:
   - DNS lookup to controlled domain
   - HTTP request to callback server
   - SMTP connection attempt
3. Send payload to target parameter
4. Monitor callback server for matching identifier
5. If callback received:
   - Confirms server-side execution (SSRF, XXE, RCE)
   - Highest confidence level (VERIFIED)
   - Records callback evidence for report
```

## 3) Per-Vulnerability-Class Strategies

### SQL Injection

| Technique | Detection Method |
|-----------|-----------------|
| Error-based | Inject SQL syntax errors → detect error messages in response |
| Union-based | UNION SELECT with incrementing columns → detect data leakage |
| Boolean-blind | True/false conditions → detect response differential |
| Time-blind | SLEEP/WAITFOR/pg_sleep → detect timing differential ≥ 1s |
| Stacked queries | Multiple statement injection → detect side effects |
| Second-order | Stored payload → trigger on subsequent operations |
| OOB | DNS/HTTP exfiltration via SQL functions |

### Cross-Site Scripting

| Technique | Detection Method |
|-----------|-----------------|
| Reflected | Payload in response body → verify executable context |
| DOM-based | Client-side analysis (V8 engine) → detect DOM sink execution |
| Stored | Submit payload → verify persistence on subsequent request |
| Context escape | Break out of HTML attribute/JS string/CSS context |
| Filter bypass | Encoding and obfuscation to evade server-side filters |

### Command Injection

| Technique | Detection Method |
|-----------|-----------------|
| Direct execution | Shell metacharacters (;, &&, \|\|, \`) → detect output |
| Blind (time) | sleep/ping delay → detect timing differential |
| OOB | DNS/HTTP callback → detect execution |

### Server-Side Request Forgery

| Technique | Detection Method |
|-----------|-----------------|
| Internal service | Request to internal IP/port → detect response content |
| Cloud metadata | Request to 169.254.169.254 → detect cloud credentials |
| Protocol smuggling | File://, gopher://, dict:// → detect protocol handling |
| DNS rebinding | DNS resolution manipulation → detect internal access |
| OOB | Callback to controlled server → confirm server-side fetch |

## 4) Negative Testing

For every mutation strategy, **negative filters** prevent false positives:

- **Known safe responses:** if a response matches the application's standard error page template, the status code change is not treated as evidence
- **Reflection without execution:** XSS payloads reflected in non-executable context (inside HTML comments, attribute values with encoding) are not counted
- **WAF detection:** if the target has a web application firewall, responses from the WAF (as opposed to the application) are identified and excluded
- **Rate limiting responses:** HTTP 429 responses are not treated as vulnerability evidence

## 5) Resource Governance

To prevent scan-induced denial of service:

| Parameter | Default | Purpose |
|-----------|---------|---------|
| Max concurrent requests | 100 | Limit parallel connections to target |
| RPS limit | 100 | Requests per second cap |
| Max goroutines | 5,000 | Resource governor ceiling |
| Queue depth | 1,000 | Pending request queue limit |
| Session timeout | 30 minutes | Maximum session duration |
| Network error rate cap | 10% | Stop if error rate exceeds threshold |
| Backend error rate cap | 20% | Stop if server error rate exceeds threshold |
| Deep sleep duration | 5 minutes | Pause period when rate limiting detected |
