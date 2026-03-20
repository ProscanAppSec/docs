# Authentication

Proscan supports multiple authentication methods to fit your organization's security requirements.

## Username and Password

Standard email/password authentication. Passwords must meet minimum complexity requirements. Accounts are locked after repeated failed login attempts with progressive delay.

## Multi-Factor Authentication (MFA)

TOTP-based multi-factor authentication using any standard authenticator app (Google Authenticator, Authy, 1Password, etc.). Recovery codes are generated during MFA setup in case the authenticator device is lost.

MFA can be enforced for all users by an administrator.

## Single Sign-On (SSO)

Proscan supports SSO through OAuth 2.0 and OpenID Connect (OIDC). Compatible with:

- Okta
- Auth0
- Azure Active Directory
- Google Workspace
- Any OIDC-compliant identity provider

SAML support is also available for organizations that require it.

### SSO Setup

1. Go to **Settings > Authentication > SSO**
2. Select your identity provider
3. Enter the provider's configuration details (client ID, client secret, issuer URL)
4. Map identity provider groups to Proscan roles
5. Test the connection
6. Enable SSO

Once enabled, users authenticate through your identity provider instead of entering a local password.

## Session Management

- Sessions expire after a configurable period of inactivity
- Active sessions can be viewed and revoked from **Settings > Security**
- JWT-based session tokens with automatic refresh

## Rate Limiting

Authentication endpoints are rate-limited to protect against brute force attacks. After several failed attempts, the account is temporarily locked with increasing delay between allowed retries.

## Audit Logging

All authentication events are logged: successful logins, failed attempts, MFA challenges, session creation and expiration, and SSO events. Logs are available under **Settings > Audit Log** and can be exported or forwarded to your SIEM.
