# Authentication

Status: accepted on 26 November 2026. Google and Apple are the only MVP sign-in methods on both iOS and Android. NestJS owns users, linked identities, authorization, and application sessions. There are no local usernames/passwords, magic links, recovery codes, email collection requirement, or email delivery service. Any future email authentication needs a separate scope change.

## Identity and onboarding

- Identify users by an internal UUID and a unique verified provider issuer/subject pair. Allow one Google and one Apple identity per user. Never merge by matching email or public name; reject linking an identity already attached to another user with `409 IDENTITY_ALREADY_LINKED`. Account merging and ownership transfers are outside the MVP.
- Let users choose an editable Unicode display name, 2 to 40 characters after trimming, without uniqueness requirements. Reject control characters. It is public profile data, never a login identifier. Do not persist provider email or avatar metadata in the MVP.
- First sign-in creates an account transactionally under the identity uniqueness constraint. Require a display name, an explicit 18+ declaration, and acknowledgement of versioned privacy information and acceptance of terms/community rules before enabling discovery or hosting. Store declaration/acknowledgement timestamps and policy version, not date of birth or identity documents. An incomplete account can only finish onboarding, manage its sign-in methods, or delete itself.
- Explicitly link a second provider from account settings. Require fresh verification of an already linked provider and of the new identity, both within five minutes and bound to the same single-use linking transaction. Lock the user row and enforce identity uniqueness transactionally.
- Unlink only after fresh verification with the provider that will remain. Never unlink the last identity. Unlinking also revokes the provider grant where supported and removes its stored credential through a retryable protected operation. Unlinking revokes all application sessions, including the current one; sign in again with the remaining provider. Explain this before confirmation. Linking alone does not revoke other devices.

## Platform and library choices

| Concern | Accepted implementation |
| --- | --- |
| Google on iOS and Android | `@react-native-google-signin/google-signin` in an Expo native development/release build, using server client configuration and supported code/token verification |
| Apple on iOS | `expo-apple-authentication`, with a server-issued nonce and server-side token/code verification |
| Apple on Android | Apple authorization-code browser flow opened with `expo-web-browser`; HTTPS `POST /v1/auth/apple/callback` on the stable API hostname, followed by a verified app return |
| Server verification | `google-auth-library` for Google; `jose` with Apple's published JWKS for Apple and application JWTs. Exchange codes at provider HTTPS endpoints; verify issuer/audience/signature/expiry/nonce, state, and PKCE where supported |
| Device secrets | `expo-secure-store` for refresh credentials; access token in memory. No tokens in ordinary query persistence, AsyncStorage, URLs, or logs |
| Application identifiers | Production bundle/package `com.alessandroferraridev.magicsessionplanner`; staging adds `.staging`. Production scheme `magicsessionplanner`, staging `magicsessionplanner-staging` |
| Apple service identifiers | Production `com.alessandroferraridev.magicsessionplanner.web`; staging adds `.staging`. Group each service with its environment's primary Apple App ID |

Google requires native configuration/development builds; Apple's Expo SDK is iOS-only. Android browser auth is an explicit requirement. See [Expo Google sign-in](https://docs.expo.dev/guides/google-authentication/), [Expo AppleAuthentication](https://docs.expo.dev/versions/latest/sdk/apple-authentication/), [Apple web configuration](https://developer.apple.com/help/account/capabilities/configure-sign-in-with-apple-for-the-web), and [Apple authorization and app return](https://developer.apple.com/documentation/signinwithapplerestapi/request-an-authorization-to-the-sign-in-with-apple-server).

Register exact HTTPS callbacks on the assigned staging/production `vercel.app` hostnames; no purchased domain is assumed. Client IDs, Apple Team/Key IDs, signing fingerprints, credentials, and assigned hostnames are provisioning outputs, not product decisions. Registrations do not already exist. Verify both providers on both platforms before completing auth.

## Transactions, callbacks, and abuse prevention

Create a random single-use authentication transaction with a ten-minute lifetime, nonce, environment/provider allowlist, and app-generated challenge. Native exchanges/browser returns prove transaction ownership; an intercepted URL scheme alone cannot authenticate. Browser return carries only a one-time opaque completion code, valid 60 seconds and redeemable with the app-held verifier. Provider tokens and application refresh tokens never appear in URLs. Use the system browser and fixed allowlisted return routes.

Cache provider public verification keys according to provider cache/rotation guidance; never accept an unknown key without verified refresh. Use PostgreSQL-backed atomic throttles shared across serverless instances: start/exchange at most 10 attempts per ten minutes per installation and privacy-preserving IP bucket, refresh at most 60 per minute per auth session, and linking/unlinking/deletion at most five attempts per hour per user. Expire throttle buckets within 24 hours; never log credentials or raw IP/location histories. Return `429` with a bounded retry delay. Recheck current account/session state inside sensitive mutation transactions.

Follow [OWASP OAuth guidance](https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html). Pin supported compatible package versions/lockfiles during scaffolding; reviewed dependency upgrades are maintenance rather than unresolved product choices.

## Application sessions and revocation

| Policy | Accepted behavior |
| --- | --- |
| Devices | Multiple concurrent devices; settings list sessions and allow current-device/all-device logout |
| Access token | RS256 JWT, 15-minute lifetime, fixed issuer/audience, `sub`, auth session ID, issued/expiry times and `kid`; reject unexpected algorithms |
| Refresh token | Random 256-bit opaque secret, SHA-256 hash at rest, atomic single-use rotation; 30-day inactivity expiry and absolute 90-day session lifetime |
| Reuse detection | Reusing a consumed refresh token revokes its entire auth session/family. Serialize refreshes in-app; after an ambiguous refresh failure, sign in again rather than retry a consumed secret |
| Immediate enforcement | Every protected API request validates JWT plus current non-revoked session/account in PostgreSQL. Logout/deletion stop existing access tokens immediately; this is request authorization, not notification polling |
| Sensitive actions | Provider verification within five minutes for linking, unlinking, and account deletion; logout needs no new provider consent |
| Provider revocation | Verified Apple server notifications, provider invalid-grant responses, and revoked/disabled identities revoke all application sessions when observed. External Google changes are not guaranteed instantly detectable without a provider signal; the absolute session limit still applies |
| Account deletion | Fresh proof, then atomically revoke sessions, cancel active hosting and withdraw active participation before acknowledgement; grant revocation and physical purge follow asynchronously |
| Signing keys | Rotate every 90 days and on suspected compromise. Keep normal previous verification keys 24 hours; emergency removal immediately invalidates affected tokens. Keys stay deployment secrets |

Retain an encrypted provider refresh/revocation credential only where required for provider token revocation, notably Apple account deletion. Keep the encryption key outside PostgreSQL; request no unrelated provider API scopes. Provider credentials are separate from application refresh tokens. Test Apple's revoke flow before launch, including deletion when the app session originated from Google.

## Deletion and demonstration boundaries

Follow the [immediate security transaction](./data-and-api.md#immediate-security-transactions) for deletion and account suspension. A deleting profile remains disabled until purge and cannot be restored through social login; after purge a later sign-in creates a new UUID with no old access. No demo role or fixture is an authentication credential: the public signed-out demo uses a separate local adapter and never issues an API access token. Provider verification remains mandatory for every real account.

## Recovery and completion criteria

Users recover Google/Apple access through that provider. Either already linked identity resolves the same account. If access to every linked provider is lost, there is no independent app recovery or support-based identity reassignment. Explain this and offer linking the second provider in settings.

Auth completion requires working registrations, first/returning sign-in, 18+ onboarding, linking/unlinking, immediate access revocation, rotation/reuse detection, provider grant revocation, and deletion. Unit, integration, HTTP E2E and native E2E cover forged/replayed credentials, expired proofs, cross-account linking, last-provider protection, concurrent refresh, wrong-environment callbacks, and identical display names. Recorded real-provider checks on iOS/Android complement controlled issuer tests. These are implementation tasks; no authentication product decision remains open.
The API integration follows [NestJS authentication](https://docs.nestjs.com/security/authentication). Retention and credential cleanup are specified in [Operations](./operations.md#retention-schedule); test suites and release scenarios are specified in [Testing strategy](./testing-strategy.md).
