# Rule Format and Sample Rules

This document describes the rule schema used by Proscan's detection engines and provides representative sample rules. The full rule database (200,000+ SAST rules, 2,300+ IaC rules, 922 Deep checks, 34 secrets patterns) is proprietary. These samples demonstrate rule quality, structure, and coverage methodology.

---

## 1) SAST Rule Schema

Each SAST rule is a structured definition with the following fields:

```
NativeRule {
  ID          string         // Unique rule identifier (e.g., "JAVA-SEC-031")
  Title       string         // Human-readable vulnerability title
  Severity    string         // ERROR, WARNING, INFO
  Confidence  string         // HIGH, MEDIUM, LOW
  Category    string         // Rule category (e.g., "JAVA-SEC")
  CWEs        []string       // CWE identifiers (e.g., ["CWE-89"])
  OWASP       []string       // OWASP Top 10 mapping (e.g., ["A03:2021"])
  Pattern     regex          // Compiled positive detection pattern
  Negative    regex          // Optional: compiled safe-context suppression pattern
  Languages   []string       // File extensions this rule applies to
  Description string         // What the rule detects and why it matters
  Remediation string         // How to fix the vulnerability
  References  []string       // External references (CVE, CWE page, docs)
}
```

### Design Principles

- Every rule has both a **positive pattern** (what to detect) and an optional **negative pattern** (what to suppress as safe)
- Rules are **language-bound** — they only fire on files with matching extensions
- Every rule maps to at least one **CWE identifier** for compliance traceability
- **Remediation guidance** is mandatory — findings always include fix instructions

## 2) Sample SAST Rules (8 of 200,000+)

### SQL Injection (CWE-89)

```yaml
ID:          JAVA-SEC-SQLI-CREATEQUERY
CWE:         CWE-89
Severity:    ERROR
Confidence:  HIGH
Languages:   [.java]
Pattern:     entityManager\.createQuery\s*\(\s*["'].*\+\s*\w+
Negative:    createQuery\s*\(\s*["'][^"']*["']\s*\)  # pure string literal (no concat)
Description: Detects string concatenation in JPA createQuery calls, indicating
             unsanitized user input in JPQL queries.
Remediation: Use parameterized queries with named parameters —
             entityManager.createQuery("SELECT u FROM User u WHERE u.id = :id")
                          .setParameter("id", userId)
```

### Cross-Site Scripting (CWE-79)

```yaml
ID:          JAVA-SEC-XSS-WRITER
CWE:         CWE-79
Severity:    ERROR
Confidence:  HIGH
Languages:   [.java]
Pattern:     response\.getWriter\(\).*request\.getParameter
Negative:    (?i)(escapeHtml|encode|sanitize|htmlEscape)
Description: Detects direct write of request parameters to HTTP response without
             output encoding, enabling reflected XSS.
Remediation: Apply output encoding before writing to response —
             use OWASP Java Encoder: Encode.forHtml(userInput)
```

### XML External Entity (CWE-611)

```yaml
ID:          JAVA-SEC-XXE-DOCBUILDER
CWE:         CWE-611
Severity:    ERROR
Confidence:  HIGH
Languages:   [.java]
Pattern:     DocumentBuilderFactory\.newInstance\(\)
Negative:    setFeature\s*\(\s*["'].*disallow-doctype-decl
Description: Detects DocumentBuilderFactory instantiation without XXE-safe
             feature configuration. Default configuration allows external entities.
Remediation: Disable external entities —
             factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true)
```

### Command Injection (CWE-78)

```yaml
ID:          JAVA-SEC-CMDI-EXEC
CWE:         CWE-78
Severity:    ERROR
Confidence:  HIGH
Languages:   [.java]
Pattern:     Runtime\.getRuntime\(\)\.exec\s*\(.*(?:request|getParameter|input|arg)
Negative:    (?i)(whitelist|allowlist|validate|sanitize)
Description: Detects Runtime.exec() with potentially user-controlled arguments.
Remediation: Avoid Runtime.exec() with user input. Use ProcessBuilder with
             argument arrays instead of shell command strings.
```

### Path Traversal (CWE-22)

```yaml
ID:          JAVA-SEC-PATH-FILE
CWE:         CWE-22
Severity:    ERROR
Confidence:  HIGH
Languages:   [.java]
Pattern:     new\s+File\s*\(.*request\.getParameter
Negative:    (?i)(normalize|canonical|resolve|sanitizePath)
Description: Detects File constructor with user-supplied path from request parameters.
Remediation: Validate and canonicalize the path. Ensure resolved path is within
             the expected base directory using getCanonicalPath() comparison.
```

### Server-Side Request Forgery (CWE-918)

```yaml
ID:          JAVA-SEC-SSRF-URL
CWE:         CWE-918
Severity:    ERROR
Confidence:  HIGH
Languages:   [.java]
Pattern:     new\s+URL\s*\(.*(?:request|getParameter|userInput|url)
Negative:    (?i)(allowlist|whitelist|validateUrl|isAllowedHost)
Description: Detects URL instantiation with user-controlled input, enabling
             server-side request forgery to internal services.
Remediation: Validate URLs against an allowlist of permitted hosts and schemes.
             Block internal IP ranges (10.x, 172.16-31.x, 192.168.x, 127.x).
```

### Insecure Deserialization (CWE-502)

```yaml
ID:          JAVA-SEC-DESER-OIS
CWE:         CWE-502
Severity:    ERROR
Confidence:  HIGH
Languages:   [.java]
Pattern:     ObjectInputStream.*readObject\s*\(
Negative:    (?i)(ObjectInputFilter|ValidatingObjectInputStream|whitelist)
Description: Detects Java native deserialization via ObjectInputStream.readObject()
             without filtering, enabling arbitrary code execution via gadget chains.
Remediation: Use ObjectInputFilter (Java 9+) to restrict deserializable classes,
             or replace native serialization with a safe format like JSON.
```

### Open Redirect (CWE-601)

```yaml
ID:          JAVA-SEC-REDIRECT-PARAM
CWE:         CWE-601
Severity:    WARNING
Confidence:  HIGH
Languages:   [.java]
Pattern:     response\.sendRedirect\s*\(\s*request\.getParameter
Negative:    (?i)(validateRedirect|isRelative|isSafeUrl|allowedDomains)
Description: Detects HTTP redirect using unvalidated user-supplied URL parameter.
Remediation: Validate redirect targets against a domain allowlist. Reject
             absolute URLs to external domains. Prefer relative redirects.
```

## 3) Taint Analysis Source/Sink Schema

See [Taint Analysis Model](taint-analysis.md) for the full formal specification. The catalog uses these schemas:

### Source Definition

```
TaintSource {
  Lang     string    // Language restriction (".go", ".java", etc.) or "" for all
  Object   string    // Receiver/package name (e.g., "r", "request")
  Member   string    // Field or method name (e.g., "FormValue", "getParameter")
  IsField  bool      // true = field access, false = method call
  Label    string    // Human description
}
```

### Sink Definition

```
TaintSink {
  Lang        string      // Language restriction or "" for all
  Object      string      // Receiver/package (e.g., "db", "cursor")
  Function    string      // Method/function name (e.g., "Query", "exec")
  ArgIdx      int         // Which argument is dangerous (0-indexed, -1 = any)
  VulnClass   string      // Vulnerability category (sqli, cmdi, xss, etc.)
  RuleID      string      // Rule identifier for findings
  Title       string      // Finding title
  Severity    string      // ERROR, WARNING, INFO
  CWEs        []string    // CWE mappings
  OWASP       []string    // OWASP Top 10 mappings
  Remediation string      // Fix guidance
}
```

### Sanitizer Definition

```
TaintSanitizer {
  Lang     string    // Language restriction or "" for all
  Object   string    // Package/module (empty = any receiver)
  Function string    // Function name (e.g., "EscapeString", "parameterize")
  ForClass string    // Which vulnerability class it sanitizes ("" = all)
}
```

## 4) Secrets Pattern Schema

```
SecretPattern {
  ID          string    // Pattern identifier (e.g., "aws-access-key")
  Name        string    // Human name (e.g., "AWS Access Key ID")
  Pattern     regex     // Compiled detection regex
  Severity    string    // critical, high, medium
  Type        string    // Secret category (aws, github, api-key, jwt, etc.)
  Description string    // What the pattern detects
}
```

### Sample Patterns (5 of 34)

| ID | Pattern | Severity | Type |
|----|---------|----------|------|
| aws-access-key | `AKIA[0-9A-Z]{16}` | critical | aws |
| github-token | `gh[pousr]_[A-Za-z0-9_]{36}` | critical | github |
| generic-api-key | `(?i)(api[_-]?key\|apikey)['":=]\s*[A-Za-z0-9_-]{20,}` | high | api-key |
| jwt-token | `eyJ[A-Za-z0-9_-]*\.eyJ[A-Za-z0-9_-]*\.[A-Za-z0-9_-]*` | high | jwt |
| rsa-private-key | `-----BEGIN RSA PRIVATE KEY-----` | critical | private-key |

## 5) IaC Rule Schema

```
IaCRule {
  ID          string     // Rule identifier (e.g., "TF001")
  Name        string     // Rule name
  Description string     // What misconfiguration it detects
  Severity    string     // critical, high, medium, low
  Category    string     // Functional category (storage, networking, secrets, etc.)
  Platform    string     // IaC platform (terraform, kubernetes, dockerfile, cloudformation)
  Pattern     regex      // Detection regex (for regex-based rules)
  Resource    string     // Resource type (for structured rules, e.g., "aws_s3_bucket")
  Attribute   string     // Attribute to check (e.g., "acl")
  BadValues   []string   // Values that indicate misconfiguration
  Remediation string     // Fix guidance
}
```

### Sample Rules (5 of 2,300+)

| ID | Platform | What It Detects | Severity |
|----|----------|----------------|----------|
| TF001 | Terraform | S3 bucket with public-read or public-read-write ACL | critical |
| TF003 | Terraform | Security group allowing 0.0.0.0/0 inbound | high |
| K8S002 | Kubernetes | Container running in privileged mode | critical |
| DF004 | Dockerfile | Secrets (PASSWORD, API_KEY, TOKEN) in ENV instructions | high |
| CFN001 | CloudFormation | S3 bucket with PublicRead/PublicReadWrite access control | critical |
