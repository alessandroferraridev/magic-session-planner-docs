# Authentication decisions and remaining choices

Local authentication uses username and password in the NestJS backend, with no email delivery service or email-link sign-in. Registration and local login are ordinary HTTPS API calls; they do not require a deep link or an owned sending domain. Google and Apple remain in the previously confirmed scope unless the product owner chooses to remove them.

## Product decisions still needed

| Decision | Proposed default | What must be settled |
| --- | --- | --- |
| Social sign-in | Keep Google and Apple alongside local credentials | Keeping them can still require provider callbacks or URL schemes; choose local-only auth if eliminating every social callback is a requirement |
| Password recovery without email | User-held, single-use recovery codes | Decide whether codes, an already-linked social identity, or no recovery is acceptable; loss of every recovery credential cannot be solved safely by guessing ownership |
| Username and public name | Unique login username separate from display name | Decide whether the login identifier is also public, whether it can change, and how social-created accounts select one |
| Account linking | Explicit linking from an authenticated account after reauthentication | Define proof of ownership for adding a password or social identity; do not merge accounts solely because provider emails match |
| Session policy | Multiple devices, short access tokens, rotating refresh sessions | Confirm inactivity duration and revocation behavior for password change, recovery, logout, and account deletion |

No email is required for the proposed local registration flow. If email collection is introduced later, its purpose, verification, recovery use, and delivery service need a separate decision; an unverified email is not proof of account ownership.

## Proposed technical defaults

- Keep an opaque internal user UUID. Propose a case-insensitive unique username of 3 to 30 ASCII letters, digits, or underscores, separate from a Unicode display name. Normalize identifiers consistently and enforce uniqueness in the database.
- Hash passwords with a maintained Argon2id library and unique salts. Benchmark on Vercel within the free runtime's limits, using at least the selected current OWASP baseline; never use plaintext, reversible encryption, or a fast generic hash for passwords.
- Propose passphrases of 15 to 128 characters, allowing spaces, Unicode, paste, and password managers, with no arbitrary composition rules or periodic password rotation. Reject known weak passwords using a bounded local blocklist.
- Apply shared server-side login and registration throttling across serverless instances. Use generic failed-login messages and bounded failure state; do not rely only on process memory or expose secrets in logs.
- Propose 15-minute access tokens and refresh sessions expiring after 30 days of inactivity. Keep refresh rotation and reuse detection, with logout from one or all devices. These session durations remain proposals.
- Require reauthentication for sensitive credential changes. Proposed password recovery revokes existing sessions. The handling of other devices after an ordinary password change must be confirmed.

These are implementation recommendations rather than additional confirmed product rules. See [OWASP password storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) and [OWASP authentication guidance](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html).

## Recovery without a sending service

The proposed code-based flow generates high-entropy single-use codes, stores only hashes, and shows the plaintext once so the user can save it outside the app. Regeneration requires reauthentication and invalidates older codes. A successful recovery consumes one code atomically, changes the password, and revokes existing sessions. Rate-limit recovery and test simultaneous use of one code.

Implement this only if the recovery policy is accepted. Never substitute security questions, a public username, an unverified email, or an unsupported manual operator reset for proof of ownership. A social-based recovery option works only when that identity was already linked and its login has been verified.

## Mobile callbacks and navigation

Removing email-link login removes its external app-return route. It does not by itself remove the callbacks needed by social providers. Choose native SDKs where supported and document the actual platform setup; [Expo Google authentication](https://docs.expo.dev/guides/google-authentication/) requires a native development build, and [Expo AppleAuthentication](https://docs.expo.dev/versions/latest/sdk/apple-authentication/) covers iOS rather than Android. A separate supported flow is needed if Apple sign-in must remain available on Android.

Push notification taps can navigate using a session ID in a safe payload and the app's notification response handler. That navigation is independent of an email authentication link and still checks authorization before loading a precise map. Do not promise that removing one sign-in method eliminates all app navigation or social callback configuration.

## Implementation completion

Before completing the auth feature, confirm the social scope, recovery policy, identifier/public-name model, account linking, and session/revocation rules. Then implement local signup/login, password hashing and throttling, credential changes, approved recovery behavior, and any retained social flows. Cover success, wrong credentials, identifier collisions, throttling, session rotation, revocation, and cross-account access at unit, integration, and E2E levels.
