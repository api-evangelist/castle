---
name: Screen a signup with Castle Filter API
description: Score an anonymous registration event before an account exists and block bots/abuse at signup.
api: https://api.castle.io
operations:
  - "POST /v1/filter"
---

# Screen a signup with Castle

Use Castle's **Filter API** to score **anonymous / pre-authentication** events
such as registration and password reset, where no user account exists yet.

## Prerequisites
- Backend authenticates to `https://api.castle.io` with HTTP Basic auth (API
  Secret as password) — see `authentication/castle-authentication.yml`.
- The signup page runs the browser SDK and produces a fresh, single-use
  `request_token` (120s TTL).

## Steps
1. On the signup form, generate a `request_token` and post it with the form.
2. Call `POST /v1/filter` with:
   - `type: "$registration"`, `status: "$attempted"`
   - `request_token`
   - `params` (e.g. the submitted `email`)
   - `context` (ip, headers)
3. Branch on `policy.action`:
   - `allow` — create the account.
   - `challenge` — require email/CAPTCHA verification first.
   - `deny` — reject the signup (likely bot or disposable email).
4. After the account is created, switch to the **Risk API** (`POST /v1/risk`)
   for all subsequent authenticated events — see `skills/castle-protect-login.md`.
5. React to List/Policy outcomes asynchronously via webhooks
   (`asyncapi/castle-webhooks.yml`).

## Testing
Force a bot verdict in Sandbox with a Mock Request Token such as
`test|device:bot_on_linux|action:deny` (`sandbox/castle-sandbox.yml`).
