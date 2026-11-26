# Authentication decisions and remaining choices

The MVP supports Google and Apple sign-in only. There is no local username/password registration, password recovery system, email-link login, or email delivery service. NestJS verifies provider credentials, creates or resolves an internal user, and issues the application's own sessions. Email authentication is a future option if a sending domain and delivery setup are chosen later.

Public display name is profile data separate from authentication identity. No login username exists in this release. Use an internal UUID and provider issuer/subject identifiers; do not identify or merge accounts by a public name or matching email.

## Product decisions still needed

| Decision | Proposed default | What must be settled |
| --- | --- | --- |
| Account linking | Allow the user to link Google and Apple explicitly from a signed-in account after fresh provider verification | Confirm linking and unlinking rules; do not merge by matching provider emails or allow removal of the last usable sign-in method |
| Session policy | Multiple devices, 15-minute application access tokens, rotating refresh sessions with 30-day inactivity expiry | Confirm durations and revocation behavior for logout, provider unlink/revocation, and account deletion |
| Platform setup | Both confirmed providers on the agreed iOS/Android scope, native SDK where supported and a verified browser flow otherwise | Select app package/bundle identifiers, register provider clients, and validate callback origins and platform support during scaffolding |

No application email collection is required for login. Persist provider email metadata only if a defined feature needs it; it is not an identity or account-linking key. A future email sign-in method needs a separate decision about domain, verification, linking, and delivery.

## Proposed technical defaults

- Keep an opaque user UUID and editable Unicode display name. Display-name uniqueness is not an authentication requirement; selecting the same name cannot grant access to another account.
- Verify social issuer, audience, signature, expiry, and transaction nonce/state before resolving an identity. Use provider-supported authorization-code/PKCE flows where applicable, maintained SDKs, and server-side credential verification.
- Apply shared throttling to social initiation/exchange and refresh endpoints across serverless instances. Reject replayed, expired, wrong-audience, or unverified credentials and redact all credential material.
- Propose 15-minute access tokens and refresh sessions expiring after 30 days of inactivity. Keep refresh rotation and reuse detection, with logout from one or all devices. These session durations remain proposals.
- Require fresh provider verification for sensitive identity linking/unlinking. Propose revoking affected sessions on unlink, provider revocation, or account deletion; finalize this policy with the session model.

Session durations and linking defaults remain recommendations. See [OWASP OAuth guidance](https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html) and the provider documentation during implementation.

## Recovery through identity providers

Users recover access to Google or Apple through that provider's recovery process. If both identities are already linked, either verified identity can access the same application account. An unrelated identity cannot claim an existing account based on a display name, an email string, or a support request.

If access to every linked provider is lost, the MVP has no independent local recovery channel. Explain this consequence clearly and encourage linking a second provider if that policy is accepted. Do not implement local passwords, recovery codes, security questions, or email resets as an implicit fallback.

## Mobile callbacks and navigation

Social providers can still need callback or URL-scheme configuration. Choose native SDKs where supported; [Expo Google authentication](https://docs.expo.dev/guides/google-authentication/) requires a native development build, and [Expo AppleAuthentication](https://docs.expo.dev/versions/latest/sdk/apple-authentication/) covers iOS rather than Android. Validate a supported Apple browser flow and safe app return on Android rather than assuming the iOS SDK covers it.

Push notification taps navigate using a safe session ID and the app's notification response handler, with authorization before loading a precise map. This app navigation is independent of social callback setup. Test the selected social return routes and provider configuration against the no-new-domain budget; a custom purchased domain is not assumed.

## Implementation completion

Before completing auth, settle linking/unlinking and session rules and obtain the provider client registrations for the selected package/bundle IDs. Implement first-sign-in profile creation, returning social login, explicit identity linking if adopted, token rotation, logout, and deletion/revocation. Test success, cancellation, forged/replayed tokens, identity uniqueness, display-name collisions without identity confusion, throttling, revocation, and cross-account access at unit, integration, and E2E levels.
