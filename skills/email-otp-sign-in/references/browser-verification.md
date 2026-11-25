# Browser-based email verification

Use this procedure for a web sign-in that can accept a provider-signed proof of email ownership. The existing email OTP path remains the fallback. Check current browser support, trial availability and verifier-library documentation before selecting APIs. [Chrome integration guide](https://developer.chrome.com/blog/email-verification-protocol-origin-trial)

1. Configure the canonical HTTPS login origin. If the browser requires an origin trial, the new project needs its own token for that origin. Serve it on the login page when configured. Keep browser-specific challenges and nonces out of shared page caches.
2. Generate a fresh, five-minute challenge bound to the browser with a Secure, HttpOnly binding cookie. Provide the derived nonce on the hidden input with `autocomplete="email-verification-token"` alongside the ordinary email field. Treat the returned token as optional.
3. Apply the verification budget before expensive work. Use a maintained verifier for parsing, constrained network lookups and signature checks. Validate the expected email, canonical audience, browser-bound nonce, timestamps, permitted algorithms, DNS issuer delegation, issuer signature and browser key-binding signature. Put deadlines on network calls.
4. Require an eligible existing account with the necessary verified-email, role and account-status checks. Atomically reserve the nonce as consumed before creating the session. Apply the same remembered-device choice and session rules as email OTP.
5. If proof is missing, unsupported, declined or invalid, continue to the normal rate-limited send path. A valid proof should complete sign-in without sending an email.

Test a valid signed fixture, wrong audience, wrong email, stale and mismatched challenges, invalid signatures, ineligible accounts and duplicate presentations. Verify the email fallback with the trial disabled. Report a real browser/provider success separately from fixture coverage.
