# API Security

The API scanner tests your REST and GraphQL endpoints for vulnerabilities aligned with the OWASP API Security Top 10.

## What It Tests

The scanner covers 125+ checks across the OWASP API Security Top 10 (2023):

| Category | What's Tested |
|----------|--------------|
| BOLA (Broken Object Level Authorization) | Attempts to access resources belonging to other users by manipulating IDs — numeric, UUID, nested, GraphQL |
| Broken Authentication | JWT weaknesses, session handling, OAuth misconfigurations |
| Broken Object Property Level Authorization | Mass assignment, excessive data exposure in responses |
| Unrestricted Resource Consumption | Rate limiting gaps, resource exhaustion |
| BFLA (Broken Function Level Authorization) | Privilege escalation via HTTP method tampering, accessing admin functions as a regular user |
| Server-Side Request Forgery | SSRF through URL parameters and redirects |
| Security Misconfiguration | Missing headers, verbose error messages, CORS issues |
| Injection | SQL, NoSQL, command, XSS, XXE, SSTI injections through API parameters |

## Endpoint Discovery

The scanner can discover API endpoints in two ways:

- **OpenAPI/Swagger spec** — import your API definition file and the scanner tests every documented endpoint
- **Crawling** — if no spec is available, the scanner discovers endpoints by observing traffic and following links

## Authentication

API scans support authenticated testing. Provide API keys, bearer tokens, or session credentials in the scan configuration. The scanner handles token refresh automatically during long scans.

## GraphQL

For GraphQL APIs, the scanner performs introspection (where allowed), enumerates queries and mutations, and tests for common GraphQL-specific issues like query depth attacks, batching abuse, and excessive field exposure.

## Usage

1. Create a scan and select **API Security**
2. Enter the base URL of your API
3. Optionally upload an OpenAPI spec
4. Configure authentication
5. Run the scan

Results map each finding to the relevant OWASP API Security Top 10 category with the request/response evidence.
