# Email OTP Sign-in

A Codex skill for adding passwordless email sign-in to an existing app. It works with the project's auth library and email provider. It covers single-use codes, resend and verification limits, copy and autofill, remembered sessions, and optional browser-based email verification.

## Install for Codex

On a machine with Node.js, npm and Git, run this command to make the skill available across projects.

```sh
npx --yes skills add YamilG/email-otp-sign-in --agent codex --global --yes
```

For a project-only install, run the same command from the project directory and omit `--global`.

## Use it

Open a new Codex chat in the target project and ask:

> Use $email-otp-sign-in to implement passwordless admin sign-in with our existing auth and email providers.

For a focused change:

> Use $email-otp-sign-in to improve OTP copy and paste. Preserve our existing code expiry and session policy.

The [skill instructions](skills/email-otp-sign-in/SKILL.md) include the defaults and acceptance checks. The separate [browser verification guide](skills/email-otp-sign-in/references/browser-verification.md) covers participating browsers and providers.
