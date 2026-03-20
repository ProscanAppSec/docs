# SAST — Static Application Security Testing

Proscan's SAST engine analyzes source code without executing it. It reads your codebase, builds an understanding of how data flows through your application, and identifies patterns that lead to vulnerabilities.

## Languages

SAST supports 60+ programming languages, including:

Go, Python, JavaScript, TypeScript, Java, C#, C, C++, Ruby, PHP, Kotlin, Swift, Rust, Scala, Perl, R, Bash, PowerShell, Terraform, Dockerfile, YAML, JSON, XML, HTML, SQL, and more.

## Detection Methods

The scanner uses multiple analysis techniques in parallel:

- Pattern matching against known vulnerability signatures
- Abstract syntax tree (AST) analysis for structural detection
- Taint tracking that follows data from user input (sources) to dangerous operations (sinks)
- Semantic analysis that understands language-specific constructs and frameworks
- Control flow analysis to determine whether vulnerable paths are reachable
- Entropy analysis for detecting hardcoded secrets and tokens

## What It Finds

Common findings include:

- SQL injection, command injection, XSS
- Path traversal and file inclusion
- Insecure deserialization
- Hardcoded credentials and weak cryptographic usage
- Server-side request forgery (SSRF)
- Missing input validation
- Insecure configurations

Each finding is mapped to its CWE identifier and, where applicable, to compliance framework controls (OWASP, PCI DSS, NIST, etc.).

## Rule Coverage

The engine ships with over 202,000 rules covering 388 CWE categories. Rules are updated with each Proscan release.

## Incremental Scanning

For repositories under version control, SAST can run in incremental mode — analyzing only files that changed since the last scan. This keeps scan times short during active development without sacrificing coverage on full scans.

## Fix Suggestions

Every finding includes a remediation suggestion: the original code, the corrected version, and a unified diff. For well-known vulnerability patterns, these come from a curated template library. For less common issues, an AI-assisted fallback generates context-aware fix suggestions.

## Usage

1. Create a scan and select **SAST**
2. Point it at a repository (URL or uploaded archive)
3. Configure language filters if needed, or let auto-detection handle it
4. Run the scan

Results appear in real time as analysis progresses.
