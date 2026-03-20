# Secrets Detection

The secrets scanner finds credentials, API keys, tokens, and other sensitive values that have been committed to your codebase. It scans current files and optionally digs through git history to catch secrets that were committed and later removed but still exist in older commits.

## Detection Methods

- **Pattern matching** against 10,000+ known secret formats (AWS keys, GitHub tokens, Stripe keys, Slack webhooks, database connection strings, and many more)
- **Entropy analysis** to flag high-randomness strings that look like secrets even if they don't match a known pattern. Tuned to filter out common false positives like UUIDs, hashes, and test data.

## Live Verification

For certain secret types, the scanner can check whether a detected credential is still active by testing it against the provider's API. Supported providers include AWS, GitHub, GitLab, Slack, Stripe, OpenAI, Twilio, and others.

This tells you whether a leaked secret is an urgent issue (active and exploitable) or a historical artifact (already rotated).

## Git History Scanning

Secrets are often committed accidentally and then removed in a later commit. The removal doesn't help — the secret still exists in the repository's history. The secrets scanner can look through past commits at a configurable depth to find these buried credentials.

## What It Finds

Common findings include:

- Cloud provider keys (AWS, GCP, Azure)
- API tokens (GitHub, GitLab, Slack, Stripe, SendGrid, Twilio)
- Database connection strings
- Private keys and certificates
- OAuth client secrets
- AI/ML API keys (OpenAI, Anthropic, Cohere, Hugging Face)
- Generic high-entropy strings

## Usage

1. Create a scan and select **Secrets**
2. Point it at a repository
3. Optionally enable git history scanning and set the commit depth
4. Run the scan

Each finding shows the secret type, the file and line where it was found, and whether it's currently active (if verification is enabled).
