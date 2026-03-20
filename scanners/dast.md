# DAST — Dynamic Application Security Testing

DAST tests your running web applications from the outside, the same way an attacker would. It crawls pages, submits forms, manipulates parameters, and observes how the application responds.

## How It Works

1. Point the scanner at a URL
2. It crawls the application, discovering pages, forms, and API endpoints
3. It tests each discovered input with a range of attack payloads
4. Results are reported with the request/response evidence that triggered the finding

The scanner uses a headless browser to handle JavaScript-heavy applications and single-page apps (SPAs). It sees what a real browser sees, not just raw HTML.

## Scan Profiles

Multiple profiles are available depending on how thorough you need the scan to be:

- **Quick** — fast surface-level scan, good for sanity checks
- **Standard** — balanced coverage and speed for regular use
- **OWASP** — focused on the OWASP Top 10 categories
- **Aggressive** — deep crawling, more payloads, longer scan time
- **Technology-aware** — adjusts test cases based on detected tech stack

You can also create custom profiles that combine specific test categories and crawling depth.

## What It Tests

- SQL injection, NoSQL injection, command injection
- Cross-site scripting (reflected and stored)
- Server-side request forgery (SSRF)
- XML external entity injection (XXE)
- Server-side template injection (SSTI)
- Authentication weaknesses (JWT issues, session fixation, OAuth misconfigurations)
- Directory traversal and file inclusion
- Security header analysis
- TLS/SSL configuration

## Authentication

DAST can test authenticated areas of your application. Provide credentials or session tokens in the scan configuration, and the scanner maintains a logged-in session throughout the crawl.

## Subdomain Discovery

For external-facing targets, DAST can run subdomain enumeration before scanning. This maps your public attack surface before testing begins.

## Usage

1. Go to **Scans > New Scan**
2. Enter the target URL
3. Choose a scan profile
4. Optionally configure authentication and scope
5. Start the scan

Findings stream in as the scan progresses. Each one includes the HTTP request and response that demonstrated the vulnerability.
