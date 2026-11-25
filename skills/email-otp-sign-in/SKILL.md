---
name: email-otp-sign-in
description: Implement or adapt passwordless email one-time-code sign-in, including resend and verification limits, copy and autofill UX, remembered sessions, and optional browser email verification. Use for email OTP login flows or targeted changes to one of these behaviors.
---

# Email OTP sign-in

Trace a real sign-in from the form through issuance, delivery, verification, session creation, protected actions and logout. Reuse the project's auth library, transactional mail provider, shared storage, account identifiers and role checks. Match the requested scope. For a copy or template change, keep the existing authentication and session policy. For a new flow, use the following small-admin defaults unless the project specifies others.

## Issue and verify

| Control | Default |
| --- | --- |
| Code | Six cryptographically random ASCII digits, including leading zeros. Carry it as a string. |
| Lifetime | Five minutes, single use. On resend, rotate the code and accept only the latest issuance. |
| Send cooldown | One send per normalized email per 60 seconds, including the first request. |
| Send budget | Three per email and 12 per trusted client IP in 15 minutes. |
| Verify budget | Five attempts per issued code, plus five per email and 30 per trusted client IP in 15 minutes. The email and IP budgets survive resends. |

Prefer the auth library's secure OTP generator and atomic verification. For app-managed storage, bind the record to the account identifier and sign-in purpose, store a purpose-specific HMAC-SHA-256 with a server-held secret, and expire it server-side. Atomically check and consume the current issuance so two concurrent submissions cannot both establish sessions. Exclude codes, tokens and message bodies from application logs and error telemetry.

Enforce separate send and verify budgets in the server entry points before dispatch or verification. Use atomic fixed-window increments in storage shared by all instances. Confirm that any other exposed auth endpoint goes through equivalent checks. Normalize email consistently with account lookup. Derive the IP from the deployment's trusted proxy configuration and use a shared fallback bucket when unavailable. Hash identifiers in limiter keys. Tune IP limits for shared networks and traffic volume. A cooldown in the client can communicate when to retry, but the server owns the decision.

For an existing-account flow, disable implicit signup. For admin access, check the current role on protected reads and mutations. Show the same send response for unknown accounts, throttled sends and delivery errors. Give clear feedback on malformed codes and verification throttling, and let the user change email.

## Copy, paste and autofill

For changes to an existing flow, derive the displayed expiry, input length and validation pattern from the issuer’s actual policy. The examples below use the six-digit, five-minute default.

- Use a recognizable sender and a descriptive subject such as “Your Example App sign-in code.” Keep the code out of the subject and authored preview. Mail clients may still create snippets from the body.
- Send plain text and HTML versions. Put “Your verification code is 012345. It expires in 5 minutes.” in plain text. In HTML, put the same six contiguous digits on their own line as selectable text. Use CSS letter spacing for visual separation. A mail client can offer its own copy action. Manual selection and paste should always work.
- For the draft origin-bound email enhancement, send the same digits through the provider's custom header API, for example `OTP-Token: "012345"; origin="https://admin.example.com"`. Derive the origin from trusted configuration, retaining a non-default port and omitting any path. Verify the outgoing payload. If a provider strips custom headers, preserve ordinary OTP and report the limitation.
- Use one labeled `type="text"` input with `inputmode="numeric"`, `autocomplete="one-time-code"`, `pattern="[0-9]{6}"`, `maxlength="6"`, and an explicit submit action. Validate six ASCII digits again on the server. Associate the expiry hint and error with the field and announce send status. Resend and change-email controls must work when the code field is empty. Prevent duplicate pending submits. For server-rendered web apps, keep manual entry available with JavaScript disabled.

## Sessions

Default to an eight-hour absolute limit and a 30-minute idle limit. Offer an unchecked “Remember this device for 30 days” checkbox. In this profile, opting in replaces both default limits with a fixed 30 days from sign-in. Preserve stricter project requirements. Record the choice and deadlines server-side and align cookie expiry. Update activity only on an unexpired session. Concurrent requests must never revive it or extend the absolute deadline.

Use the framework's Secure, HttpOnly session cookie in production. Sign-out revokes the server session and clears the cookie. If introducing new persistence fields, define how existing sessions expire and migrate them without extending their access.

## Optional browser verification

When the requested flow includes verification without an emailed code, read [the browser verification procedure](references/browser-verification.md). This is a separate integration from both the email header and input autofill. Keep the rate-limited email path usable when the browser or provider cannot verify the address.

## Prove the flow

Choose checks for the requested changes and the session modes the project supports. Use the project's normal entry points, existing test tools and a controlled test mailbox or mail capture.

- Exercise a leading-zero code through the rendered message and form. Inspect the actual mail payload, both bodies and header. Distinguish manual copy/paste, OS autofill and a mail client's native copy shortcut in the results.
- Check correct, expired, consumed and superseded codes. Exercise the send cooldown and verification budgets across resends, shared storage and concurrent submissions. Check that an unknown address gets the generic response.
- Verify access to a protected page, reload, enabled session modes, applicable idle and absolute expiry, server-side role denial and logout. Use a controlled clock for long deadlines.
- Report settings, migrations, observed behavior and client/provider-specific features still unverified. Scope live email, trial enrollment and deployment to the user's request.
