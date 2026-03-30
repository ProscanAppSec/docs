# Taint Analysis Model

This document provides a formal specification of Proscan's intra-procedural taint propagation engine. It describes the analysis model, source/sink/sanitizer taxonomy, and flow confidence calculation without exposing the full catalog or implementation.

---

## 1) Analysis Scope

| Property | Value |
|----------|-------|
| Analysis type | Intra-procedural (within function boundaries) |
| Maximum call depth | 5 |
| Maximum file size | 200 KB |
| Source catalog | 136 registered sources |
| Sink catalog | 84 registered sinks |
| Sanitizer catalog | 205 registered sanitizers |
| Languages with taint support | Go, Python, JavaScript, Java, C#, Ruby, PHP, Kotlin, TypeScript, Rust, Swift |
| Vulnerability classes tracked | 12 (see below) |
| Minimum flow confidence | 0.50 (flows below this are dropped) |

## 2) Vulnerability Classes

The taint engine tracks flows for these vulnerability categories:

| Class | Description | Primary CWE |
|-------|-------------|-------------|
| sqli | SQL injection | CWE-89 |
| cmdi | OS command injection | CWE-78 |
| xss | Cross-site scripting | CWE-79 |
| ssrf | Server-side request forgery | CWE-918 |
| path_traversal | Path/directory traversal | CWE-22 |
| redirect | Open redirect | CWE-601 |
| deserialization | Insecure deserialization | CWE-502 |
| xxe | XML external entity | CWE-611 |
| ldapi | LDAP injection | CWE-90 |
| nosqli | NoSQL injection | CWE-943 |
| ssti | Server-side template injection | CWE-94 |
| log_injection | Log injection/forging | CWE-117 |

## 3) Formal Model

### 3.1 Definitions

- **Source:** an expression that reads external (user-controlled) input into a program variable. Examples: HTTP request parameters, form data, headers, cookies, URL components, file reads, environment variables.

- **Sink:** a function call that performs a security-sensitive operation. Examples: SQL query execution, OS command execution, HTTP response output, file system operations, network requests, redirect calls.

- **Sanitizer:** a function that transforms tainted data into a safe form for a specific vulnerability class. Examples: parameterized query binding, HTML encoding, URL validation, path canonicalization.

- **Tainted variable:** a variable whose value is derived (directly or transitively) from a source without passing through a sanitizer for the relevant vulnerability class.

### 3.2 Propagation Rules

```
Rule 1 — Source Introduction:
  If statement S matches source s ∈ SourceCatalog:
    Mark LHS variable as TAINTED
    Record (variable, source_line, source_label)

Rule 2 — Assignment Propagation:
  If statement S assigns variable V from expression containing
  tainted variable T:
    Mark V as TAINTED
    Inherit taint metadata from T

Rule 3 — Framework Annotation Pre-population:
  If function signature (within 5 lines above body start) contains
  framework annotations:
    @RequestParam, @PathVariable, @RequestBody, @RequestHeader,
    @CookieValue, @MatrixVariable
  Then: mark all annotated parameters as TAINTED before body analysis

Rule 4 — Return Propagation:
  If function F returns a tainted variable:
    Mark F as ReturnsTaint = true
  If statement S calls function F where F.ReturnsTaint = true:
    Mark LHS variable as TAINTED

Rule 5 — Sink Detection:
  If statement S matches sink k ∈ SinkCatalog:
    If argument at k.ArgIdx is a tainted variable:
      Check for sanitizer (Rule 6)
      If no sanitizer found → emit taint flow finding

Rule 6 — Sanitizer Check:
  For the code region between source_line and sink_line:
    If any statement matches sanitizer z ∈ SanitizerCatalog
    where z.ForClass matches k.VulnClass (or z.ForClass = ""):
      Mark flow as SANITIZED
      Do not emit finding
```

### 3.3 Literal Argument Exclusion

If the sink argument is a string literal, numeric literal, or boolean constant (not a variable), the flow is not emitted. Literal arguments represent hardcoded values that are not user-controlled.

### 3.4 SQLi-Specific Sanitization Heuristic

For SQL injection sinks, the engine also checks for parameterization evidence on the sink line:
- Presence of `?` placeholder
- Presence of `$1`, `:1`, `@param` named parameters
- Presence of `Prepare` or `parameterize` keywords

If parameterization evidence is found, the flow is treated as sanitized.

## 4) Confidence Calculation

Taint flow confidence starts at **0.75** (base) and is adjusted:

| Factor | Effect |
|--------|--------|
| Framework annotation source | +0.10 (high certainty of user input) |
| Direct source match (exact API) | +0.05 |
| Multiple propagation steps | −0.05 per step after 3rd |
| Sink with specific ArgIdx | +0.05 (vs. ArgIdx = -1 / any) |
| Near sanitizer (partial match) | −0.10 |

Final confidence is clamped to [0.50, 1.00]. Flows below 0.50 are dropped.

## 5) Source Categories

Sources are organized by input vector:

| Category | Examples | Languages |
|----------|----------|-----------|
| HTTP request parameters | FormValue, getParameter, query, params | All web |
| HTTP request body | Body, InputStream, request.body | All web |
| HTTP headers | Header, getHeader, headers | All web |
| URL components | URL, Path, RequestURI, url | All web |
| Cookies | Cookie, getCookies, cookies | All web |
| Form data | Form, PostForm, FILES | All web |
| Command line | os.Args, sys.argv, ARGV | Go, Python, Ruby |
| Environment | os.Getenv, os.environ, ENV | Go, Python, Ruby |
| File reads | os.ReadFile, open().read() | Go, Python |
| Database reads | Rows.Scan, fetchone, cursor | Go, Python, Java |

## 6) Sink Categories

Sinks are organized by vulnerability class:

| Vulnerability | Sink Examples | Risk |
|---------------|--------------|------|
| SQL injection | db.Query, db.Exec, cursor.execute, createQuery, prepareStatement | Data exfiltration, data manipulation |
| Command injection | exec.Command, os.system, subprocess.run, Runtime.exec | Arbitrary code execution |
| XSS | response.Write, print(), document.write, innerHTML | Session hijacking, defacement |
| SSRF | http.Get, urllib.request, fetch, new URL | Internal service access |
| Path traversal | os.Open, new File, open(), fs.readFile | Unauthorized file access |
| Open redirect | Redirect, sendRedirect, redirect_to | Phishing, credential theft |
| XXE | XMLParser, DocumentBuilder, createXMLStreamReader | File disclosure, SSRF |
| Deserialization | readObject, pickle.loads, yaml.load | Remote code execution |

## 7) Sanitizer Recognition

The engine recognizes sanitizers by function name and vulnerability class binding:

| Category | Universal Names | Language-Specific Examples |
|----------|----------------|--------------------------|
| General | escape, sanitize, encode, validate, clean, filter | — |
| XSS | htmlEscape, encodeForHTML | Go: html.EscapeString, template.HTMLEscapeString |
| SQL | parameterize, bindParam | — (parameterization detected via heuristic) |
| Path traversal | canonicalize, normalize | Go: filepath.Clean; Python: os.path.realpath |
| URL | validateUrl, isAllowedHost | Go: url.QueryEscape, url.PathEscape |
| Command | shellescape, shlex.quote | — |

## 8) Limitations

- Analysis is **intra-procedural** (within function boundaries). Cross-file and cross-function taint tracking requires call graph construction, which is not yet implemented for all languages.
- Maximum call depth of 5 limits analysis of deeply nested call chains.
- Framework annotation detection currently covers Spring/JAX-RS style annotations. Other frameworks may require manual source registration.
- Sanitizer recognition is name-based. Custom sanitizers with non-standard names must be registered in the catalog to be recognized.
