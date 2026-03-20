# ProScan Deep

ProScan Deep is the most advanced scanner in the platform. It goes beyond traditional DAST by running a multi-phase scan workflow with dozens of specialized detectors, and it includes a full penetration testing toolkit built directly into the interface.

## Multi-Phase Scanning

A ProScan Deep scan runs through 13 phases automatically:

1. Connectivity and pre-checks (error page detection, WAF identification, technology fingerprinting)
2. URL crawling and sitemap construction
3. Parameter and input discovery
4. Technology-specific CVE checks
5. Injection and vulnerability testing
6. Payload generation and delivery
7. Per-directory and per-file analysis
8. Technology refinement and confirmation
9. Technology-specific deep tests
10. Cross-phase analysis and false positive elimination

Each phase builds on the previous one. The scanner adapts its approach based on what it discovers about the target.

## Detection Coverage

- 72 specialized detector modules
- 1,100+ vulnerability checks
- WAF detection and evasion techniques
- Evidence-based verification with confidence scoring on every finding

## Pen Test Toolkit

ProScan Deep includes six tools that security engineers would typically need separate software for:

| Tool | Purpose |
|------|---------|
| **Proxy** | Intercept and inspect HTTP/HTTPS traffic between your browser and the target |
| **Repeater** | Modify and resend individual requests to test different inputs |
| **Intruder** | Automate payload delivery across parameters — fuzzing, brute force, enumeration |
| **Decoder** | Encode, decode, and transform data (Base64, URL, hex, HTML entities, etc.) |
| **Comparer** | Diff two responses side by side to spot behavioral differences |
| **Sequencer** | Analyze the randomness of tokens and session identifiers |

These are accessible from within the ProScan Deep interface. No external tools or additional licenses needed.

## When to Use ProScan Deep

Use ProScan Deep when you need more than a standard DAST scan — for pre-release security assessments, internal penetration tests, or deep testing of critical applications. For routine scanning across many targets, standard DAST is faster and more appropriate.

## Usage

1. Go to **ProScan Deep** from the main navigation
2. Enter a target URL
3. Configure scan options (scope, authentication, evasion settings)
4. Start the scan

During and after the scan, you can use the pen test tools to investigate findings further, replay attacks, or probe specific areas of the application manually.
